---
slug: writing-ruby
categories: [languages]
priority: 1
description: Ruby 4.0+ style -- expressive objects, Enumerable, keyword arguments, exceptions, immutable value objects, testing.
applies_when:
  - writing Ruby code
  - reviewing Ruby code
  - designing Ruby APIs
  - handling errors in Ruby
related: [coding-style, writing-tests]
source: https://docs.ruby-lang.org/en/4.0/
---

# Ruby

## Object Design

Prefer small objects with explicit public APIs over procedural scripts or bags of hashes. Ruby's flexibility is a tool for clarity, not permission to hide contracts.

```ruby
# WRONG: untyped bag of keys
def invoice_total(invoice)
  invoice[:line_items].sum { |item| item[:quantity] * item[:unit_price] }
end

# CORRECT: explicit object boundary
class Invoice
  def initialize(line_items:)
    @line_items = line_items
  end

  def total
    line_items.sum(&:subtotal)
  end

  private

  attr_reader :line_items
end
```

Rules:
- Use plain Ruby objects for domain behavior; do not make everything a module function.
- Keep instance variables private. Expose behavior, not storage.
- Prefer constructor keyword arguments for required named state.
- Use `attr_reader` for simple read access; avoid `attr_accessor` unless mutation is part of the object contract.
- Do not monkey-patch core classes in application code. Use refinements only when the boundary is narrow and explicit.

## Value Objects

Use `Data.define` for immutable value objects that mostly carry named fields. Use a normal class when construction needs validation, invariants, custom equality, or meaningful behavior beyond stored attributes.

```ruby
Money = Data.define(:amount_cents, :currency) do
  def +(other)
    raise ArgumentError, "currency mismatch" unless currency == other.currency

    self.class.new(amount_cents + other.amount_cents, currency)
  end
end
```

Rules:
- Prefer immutable value objects for identifiers, money, date ranges, parsed configuration, and command inputs.
- Validate external input at the boundary before building value objects.
- Avoid passing raw `Hash` values through multiple layers; convert once into a named object.
- Use `Struct` only when mutability is intended; otherwise prefer `Data.define`.

## Enumerable First

Use `Enumerable` and collection pipelines for transformations, but stop before cleverness harms debugging.

```ruby
# WRONG: manual accumulator for a simple filter/map
emails = []
users.each do |user|
  emails << user.email if user.active?
end

# CORRECT: intention is visible
emails = users.select(&:active?).map(&:email)
```

Rules:
- Use `map`, `filter`/`select`, `reject`, `find`, `any?`, `all?`, `none?`, `partition`, `tally`, and `group_by` before writing loops.
- Use `filter_map` when selection and transformation are the same step.
- Use `each_with_object` for building hashes or arrays with state.
- Avoid long chains with multiple side effects. Break them into named intermediate values.
- Use `lazy` only for genuinely large or infinite sequences.

## Error Handling

Raise exceptions for failures the caller cannot treat as ordinary values. Return explicit result objects or `nil` only when absence is an expected domain outcome.

```ruby
# WRONG: rescues everything and hides the cause
def load_config(path)
  YAML.load_file(path)
rescue
  {}
end

# CORRECT: narrow rescue, preserve context
def load_config(path)
  YAML.safe_load_file(path, permitted_classes: [], aliases: false)
rescue Psych::SyntaxError => error
  raise ConfigError, "invalid YAML in #{path}: #{error.message}"
end
```

Rules:
- Rescue specific exception classes, never bare `rescue` in business logic.
- Keep rescue blocks narrow. Do not wrap a whole method if only one call can fail.
- Define a package- or feature-level base error, then subclass for caller-relevant cases.
- Use `raise CustomError, message` with useful context; do not raise strings from deep code.
- Use bang methods for variants that raise (`save!`, `fetch!`) and non-bang methods for variants that return an expected falsey/result value.
- Let unexpected exceptions crash to the framework or job runner; do not log-and-swallow.

## Pattern Matching

Use pattern matching for structured branching over arrays, hashes, and value objects when it makes shape constraints obvious.

```ruby
case payload
in { "type" => "user.created", "data" => { "id" => id, "email" => email } }
  CreateUser.call(id:, email:)
in { "type" => type }
  raise UnknownEventType, type
else
  raise InvalidPayload, "missing event type"
end
```

Rules:
- Use pattern matching when the shape matters; use ordinary conditionals for simple booleans.
- Prefer named captures over indexing into arrays or nested hashes.
- Keep patterns small. Extract complicated guards into predicate methods.
- Do not use pattern matching to avoid designing a real type when the shape crosses a boundary repeatedly.

## Keyword Arguments

Use keyword arguments for public APIs with more than one non-obvious argument. Avoid boolean parameters and positional argument soup.

```ruby
# WRONG: call site is opaque
notify(user, true, 3)

# CORRECT: call site explains itself
notify(user, urgent: true, retry_limit: 3)
```

Rules:
- Use required keywords for required options: `def initialize(name:, email:)`.
- Use optional keywords with defaults only when the default is safe and unsurprising.
- Replace boolean keywords with semantic symbols or separate methods when behavior meaning is not obvious.
- Use `**kwargs` sparingly and only for intentional forwarding.

## Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Classes, modules | PascalCase | `HttpClient` |
| Methods, variables | snake_case | `parse_config` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_RETRIES` |
| Files | snake_case | `http_client.rb` |
| Predicate methods | trailing `?` | `valid?`, `empty?` |
| Raising or mutating variants | trailing `!` | `save!`, `normalize!` |
| Conversion methods | `to_*` / `as_*` | `to_h`, `as_json` |

Rules:
- Predicate methods should return real booleans when callers will branch on them.
- Use `?` only for questions, not for methods that perform commands.
- Use `!` only when there is a paired safer method or the method is surprisingly dangerous.
- Avoid abbreviations unless they are dominant in the domain.

## Blocks and Methods

Blocks are Ruby's control-flow extension point. Use them deliberately for resource scope, callbacks, transactions, and collection operations.

```ruby
File.open(path) do |file|
  import(file)
end
```

Rules:
- Use `{ ... }` for single-line blocks and `do ... end` for multi-line blocks.
- Prefer `yield` for simple block APIs; accept `&block` only when you need to store or forward it.
- Use endless methods only for trivial expression bodies; avoid them when conditionals, rescue, or multiline formatting improves clarity.
- Do not use `and` / `or` for normal conditionals; use `&&` / `||` because precedence is clearer.

## Type Information

Ruby is dynamic, but production code still needs contracts.

Rules:
- Make contracts obvious through names, constructor keywords, value objects, and focused methods.
- Add YARD docs or RBS/Sorbet types for public gem APIs, complex service boundaries, or areas with frequent misuse.
- Prefer runtime validation at external boundaries over trusting hashes from HTTP, JSON, YAML, CLI arguments, or environment variables.
- Keep metaprogramming small and documented; generated methods should be discoverable by tests and search.

## Testing

Use the project's existing test framework. For greenfield Ruby libraries, prefer Minitest for small standard-library-friendly projects and RSpec when the surrounding ecosystem already uses it.

```ruby
RSpec.describe Invoice do
  describe "#total" do
    it "sums line item subtotals" do
      invoice = Invoice.new(line_items: [
        LineItem.new(quantity: 2, unit_price_cents: 500),
        LineItem.new(quantity: 1, unit_price_cents: 250)
      ])

      expect(invoice.total_cents).to eq(1_250)
    end
  end
end
```

Rules:
- Test public behavior, not private methods.
- Use factories or builders for noisy setup; keep each example's distinguishing data local.
- Prefer `expect { action }.to change { value }` for side effects.
- Test error cases and boundary parsing, not only happy paths.
- Run `bundle exec rubocop` and the test suite before shipping Ruby changes.

## Bundler and Project Hygiene

Rules:
- Commit `Gemfile.lock` for applications. For gems, follow the project's convention.
- Use `bundle exec` when invoking project-local tools.
- Keep development/test gems in the correct Bundler groups.
- Avoid adding dependencies for one-line helpers; Ruby and Active Support often already provide the primitive.
- Pin minimum Ruby versions intentionally in gemspecs and CI.

## References

- Ruby 4.0 release notes: https://www.ruby-lang.org/en/news/2025/12/25/ruby-4-0-0-released/
- Ruby 4.0 docs: https://docs.ruby-lang.org/en/4.0/
- Ruby pattern matching docs: https://docs.ruby-lang.org/en/4.0/syntax/pattern_matching_rdoc.html
- Ruby `Data` docs: https://docs.ruby-lang.org/en/4.0/Data.html
- Ruby Style Guide: https://rubystyle.guide/
- RuboCop docs: https://docs.rubocop.org/rubocop/latest/
