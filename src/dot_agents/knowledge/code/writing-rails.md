---
slug: writing-rails
categories: [languages]
priority: 1
description: Ruby on Rails 8.1+ style -- convention-first MVC, Active Record boundaries, Hotwire, jobs, callbacks, security, tests.
applies_when:
  - writing Ruby on Rails code
  - reviewing Rails MVC changes
  - designing Active Record models
  - testing Rails applications
related: [writing-ruby, writing-tests, testing-pyramid]
source: https://guides.rubyonrails.org/
---

# Ruby on Rails

## Rails Defaults First

Rails is most productive when the app follows its conventions. Reach for ordinary models, controllers, jobs, mailers, views, migrations, fixtures/factories, and concerns before inventing a parallel architecture.

Rules:
- Keep Rails defaults unless the app has a concrete reason to diverge.
- Put code where Rails will autoload it by convention; avoid custom require/load paths.
- Prefer framework primitives before service-framework gems, command buses, custom dependency injection, or homegrown ORMs.
- Keep `config.load_defaults` current during upgrades, then handle changed defaults explicitly.
- Use `bin/rails` and `bin/rake` binstubs so commands run in the app's Bundler context.

## Models

Active Record models should express persistence, associations, validations, and domain behavior tied to the record. They should not become unrelated workflow scripts.

```ruby
class Order < ApplicationRecord
  enum :status, { pending: 0, paid: 1, shipped: 2, canceled: 3 }

  belongs_to :account
  has_many :line_items, dependent: :destroy

  validates :number, presence: true, uniqueness: true
  validates :status, presence: true

  def total_cents
    line_items.sum(&:subtotal_cents)
  end
end
```

Rules:
- Put macro-style declarations near the top: constants, attributes, enums, associations, validations, callbacks, scopes.
- Prefer explicit hash-backed enums; array-backed enums make database values depend on ordering.
- Use `has_many :through` when the relationship may need validations, attributes, or behavior.
- Use `dependent:` deliberately on every destructive association.
- Use non-Active Record models with `ActiveModel::Model` for form objects and validation-only objects.
- Move workflows spanning many aggregates into plain Ruby objects under a conventional namespace such as `app/services`, but keep them small and boring.

## Controllers

Controllers coordinate HTTP: load records, authorize, call one operation, choose a response. Keep business logic out of actions.

```ruby
class OrdersController < ApplicationController
  before_action :set_order, only: %i[show update]

  def update
    if @order.update(order_params)
      redirect_to @order, notice: "Order updated"
    else
      render :edit, status: :unprocessable_entity
    end
  end

  private

  def set_order
    @order = current_account.orders.find(params[:id])
  end

  def order_params
    params.expect(order: %i[reference ship_on])
  end
end
```

Rules:
- Prefer RESTful resource controllers over custom action collections.
- Keep actions short. One lookup, one command/update, one response path.
- Use symbolic HTTP statuses: `:unprocessable_entity`, `:not_found`, `:created`.
- Scope record lookup through the current account/user when multi-tenancy or authorization matters.
- Use strong parameters (`expect` / `permit`) at the controller boundary; never mass-assign raw `params`.
- Do not put formatting helpers or query construction in controllers.

## Routes

Routes should expose resource shape clearly.

```ruby
resources :projects do
  resources :tasks, shallow: true
end

namespace :admin do
  resources :users
end
```

Rules:
- Prefer `resources` and `resource` over hand-written verb/path/action routes.
- Use `member` and `collection` for extra REST-adjacent actions.
- Avoid nesting deeper than one level; use `shallow: true` when child records have their own identity.
- Namespace admin, API, and internal surfaces explicitly.
- Never add broad wildcard routes that expose arbitrary controller actions.

## Active Record Queries

Use relations as composable query objects. Keep SQL injection-safe APIs and make query intent visible.

```ruby
class Invoice < ApplicationRecord
  scope :overdue, -> { where(status: :open).where("due_on < ?", Date.current) }
  scope :for_account, ->(account) { where(account:) }
end

Invoice.for_account(current_account).overdue.order(due_on: :asc)
```

Rules:
- Use hash conditions and placeholders; never interpolate user input into SQL strings.
- Use scopes for reusable query fragments, but keep them composable and side-effect-free.
- Use `includes`/`preload`/`eager_load` intentionally to fix N+1 queries after checking access patterns.
- Use `find_each` / `in_batches` for batch work.
- Use `pluck`, `pick`, and `ids` when you need scalar values, not model instances.
- Use `size` when a loaded association may already know its count; use `count` when you need a database count.

## Transactions and Side Effects

Keep database state changes atomic, and defer external side effects until commit.

```ruby
Order.transaction do
  order.pay!(payment)
  order.line_items.lock.each(&:reserve!)
end

OrderMailer.receipt(order).deliver_later
```

Rules:
- Wrap multi-record state changes in transactions.
- Use `lock` / `with_lock` when concurrent writes can violate invariants.
- Do not send emails, enqueue externally visible jobs, call webhooks, or publish events from inside a transaction unless the side effect is explicitly commit-aware.
- Prefer `after_commit` over `after_save` for side effects that depend on durable database state.
- Use bang persistence (`save!`, `update!`, `create!`) inside transactions so failure rolls back.

## Callbacks

Callbacks are good for local model invariants and lifecycle bookkeeping. They are dangerous for hidden workflows.

Rules:
- Use callbacks for normalization, derived fields, counters, and model-local lifecycle concerns.
- Avoid callbacks that perform remote IO, enqueue broad workflows, or mutate unrelated aggregates.
- Keep callback methods private.
- Do not call `save`, `update`, or other persistence methods inside callbacks; assign attributes before persistence or use commit callbacks.
- Prefer explicit service/workflow objects when the caller should know that an action happens.

## Migrations and Schema

Migrations are production operations, not just schema diffs.

Rules:
- Prefer reversible `change`; use `up`/`down` when reversibility is not obvious.
- Add database constraints for invariants that must survive race conditions: `null: false`, unique indexes, foreign keys, check constraints.
- Backfill large tables with care: deploy additive nullable columns first, backfill in batches, then add constraints.
- Avoid using application models directly in migrations; define a minimal migration-local class when data migration needs Active Record.
- Keep `db/schema.rb` or `db/structure.sql` authoritative and committed according to the app's convention.
- Name indexes and foreign keys when the generated names are too long or unclear.

## Views, Hotwire, and JavaScript

Use server-rendered HTML and Hotwire as the default interaction model unless the product clearly needs a heavier client app.

Rules:
- Keep ERB views simple. Move formatting to helpers, presenters, or model methods with real domain meaning.
- Use partials for repeated fragments; pass locals instead of relying on many instance variables.
- Use Turbo Frames for independently replaceable page regions.
- Use Turbo Streams for server-driven updates after mutations or broadcasts.
- Use Stimulus controllers for local client behavior; keep them small and HTML-driven.
- Avoid duplicating validation and state machines in JavaScript when the server owns the truth.

## Background Jobs

Use Active Job for work that can be retried, delayed, or moved out of the request path.

```ruby
class SendReceiptJob < ApplicationJob
  queue_as :mailers

  def perform(order)
    OrderMailer.receipt(order).deliver_now
  end
end
```

Rules:
- Pass GlobalID-backed records or simple primitives, not large serialized objects.
- Make jobs idempotent; retries should not double-charge, double-send, or corrupt state.
- Use queue names intentionally for latency and capacity.
- Keep long-running workflows resumable or checkpointed when failures are expected.
- Test enqueue behavior separately from job side effects.

## Security

Rails provides strong defaults, but they only work when code stays inside the framework's safe paths.

Rules:
- Use strong parameters for all mass assignment.
- Use Rails form helpers and escaping; avoid `html_safe` unless the string was sanitized or generated by trusted code.
- Use parameterized queries or Arel for SQL fragments.
- Store secrets in credentials or environment-backed secret managers, not constants or YAML committed to the repo.
- Scope authorization and tenant access in queries, not just views.
- Keep CSRF protections enabled for browser sessions.
- Use signed/encrypted cookies and avoid storing sensitive data client-side unless it is encrypted and minimal.

## Testing

Follow the app's test framework. Rails defaults to Minitest, but many apps use RSpec; consistency beats preference.

Rules:
- Test models for validations, associations that affect behavior, scopes, and domain methods.
- Test requests/controllers at the HTTP boundary: params, auth, status, redirects, rendered errors.
- Test system flows for critical user journeys with Capybara or the app's established system-test setup.
- Use factories or fixtures consistently; do not mix styles casually.
- Freeze time for time-dependent behavior using Rails helpers.
- Assert enqueued jobs, delivered mail, broadcasts, and database changes explicitly.
- Keep tests parallel-safe: avoid global state, shared records with hidden ordering, and time-based assumptions.

## Observability and Production

Rules:
- Use `Rails.logger` with structured context where the app supports it.
- Subscribe to or instrument meaningful `ActiveSupport::Notifications` events for important workflows.
- Treat N+1 queries, slow queries, job retries, and cache hit rates as production concerns.
- Configure error reporting intentionally; do not rescue exceptions just to log them.
- Use caching with explicit keys and invalidation rules. Prefer Russian-doll caching only when object lifetimes are clear.

## References

- Rails Guides 8.1.3: https://guides.rubyonrails.org/
- Active Record Basics: https://guides.rubyonrails.org/active_record_basics.html
- Active Record Query Interface: https://guides.rubyonrails.org/active_record_querying.html
- Active Record Callbacks: https://guides.rubyonrails.org/active_record_callbacks.html
- Action Controller Overview: https://guides.rubyonrails.org/action_controller_overview.html
- Rails Security Guide: https://guides.rubyonrails.org/security.html
- Rails Style Guide: https://rails.rubystyle.guide/
