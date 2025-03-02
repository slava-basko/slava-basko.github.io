---
layout: post
---

Not so long ago, I joined a logistics project with quite a legacy codebase. 
Eventually, we updated the project to PHP 7 and then to PHP 8. 
That wasn’t a big deal since there were literally no dependencies.

The real struggle was maintaining business rules.
Basically, a lot of `if {} else {}` statements, lists, code duplication, etc.
These rules weren’t fixed, some rules work fine today, but tomorrow conditions change, and we need to adjust the code.
For example, today, the EU is one list of countries, but after Brexit, it’s another one.

Things got even messier because we spoke with clients in different languages. 
They called certain conditions/rules one way, and we called them another. 
It was clear we needed a common language that both sides could understand.

That’s when I remembered the beautiful and simple **Specification Pattern** — a perfect fit for our problem. 
Plus, it’s super handy even between developers.

## Building a Specification Library

I wanted to introduce specifications, so I started looking for a good and simple PHP library. 
Unfortunately, everything I found was either overengineered or didn’t meet our criteria — PHP 5.5 at the time, no dependencies.

So, I built my own, focusing on simplicity, high composability, and avoiding fancy modern PHP features that wouldn’t 
work on older PHP versions and didn’t bring much benefit anyway.

👉 You can check it out on GitHub: [https://github.com/slava-basko/specification-php](https://github.com/slava-basko/specification-php).

## Why Use Specifications?

At first, it might seem like overkill. But trust me, the value becomes clear over time.
Take a simple example: **Shipment goes to the UK**.
You could implement a `Shipment::toUK(): bool` method, but that quickly becomes messy. You’ll end up with a ton of helper methods cluttering your entity. A better approach is to encapsulate this logic in a specification:
```php
class ShipmentToUk extends AbstractSpecification {
    /**
     * @param Shipment $candidate
     * @return bool
     */
    public function isSatisfiedBy($candidate)
    {
        return $candidate->getDesctination() == 'UK';
    }
}
```
That worked for a while, but then the client said:
> Well, we consider Jersey and the Isle of Man as part of the UK too.

No problem! Just tweak the specification:
```php
class ShipmentToUk extends AbstractSpecification {
    /**
     * @param Shipment $candidate
     * @return bool
     */
    public function isSatisfiedBy($candidate)
    {
        return in_array($candidate->getDestination(), ['UK', 'GB', 'JE', 'IM']);
    }
}
```
Boom! One small change, and it applies everywhere. 
Plus, now everyone understands what **Shipment goes to the UK** actually means.

## Composing Specifications

Next, let’s introduce a **High-value UK shipment** specification.
First, we define a **High-value** specification:
```php
class HighValue extends AbstractSpecification {
    public function __construct(private CurrencyConverter $currencyConverter) {}

    /**
     * @param Shipment $candidate
     * @return bool
     */
    public function isSatisfiedBy($candidate)
    {
        return $this->currencyConverter->toGBP($candidate->getValue()) > 300;
    }
}
```
Then, we combine it with **Shipment goes to the UK**:
```php
$highValueUkShipmentSpecification = new AndSpecification([
    new ShipmentToUk(),
    new HighValue(new CurrencyConverter())
]);
```

## The Real Benefit

Now, imagine your system is using these specifications everywhere, and the client suddenly says:
> We noticed the system doesn’t recognize Guernsey as part of the UK. We need to fix this.

No problem at all! You only need to update **Shipment goes to the UK**:
```php
return in_array($candidate->getDestination(), ['UK', 'GB', 'JE', 'IM', 'GG']);
```
Done! Changes reflected on **Shipment goes to the UK** specification as well on **High-value UK shipment**.

## Final Thoughts

The Specification Pattern keeps business rules clean, encapsulated, and easy to adjust. 
No more hunting for messy `if {} else {}` statements. 
Just tweak the relevant spec, and the change propagates across the system.

Give it a try — you’ll thank yourself later. 😎
