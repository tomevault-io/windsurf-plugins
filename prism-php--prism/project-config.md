---
trigger: always_on
description: 'api_key' => env('GEMINI_API_KEY', ''),
---

# Gemini

## Configuration

```php
'gemini' => [
    'api_key' => env('GEMINI_API_KEY', ''),
    'url' => env('GEMINI_URL', 'https://generativelanguage.googleapis.com/v1beta/models'),
],
```

## Search grounding

Google Gemini offers built-in search grounding capabilities that allow your AI to search the web for real-time information. This is a provider tool that uses Google's search infrastructure. For more information about the difference between custom tools and provider tools, see [Tools & Function Calling](/core-concepts/tools-function-calling#provider-tools).

You may enable Google search grounding on text requests using withProviderTools:

```php
use Prism\Prism\Facades\Prism;
use Prism\Prism\Enums\Provider;
use Prism\Prism\ValueObjects\ProviderTool;

$response = Prism::text()
    ->using(Provider::Gemini, 'gemini-2.0-flash')
    ->withPrompt('What is the stock price of Google right now?')
    // Enable search grounding
    ->withProviderTools([
            new ProviderTool('google_search')
        ])
    ->asText();
```

If you use search groundings, Google require you meet certain [display requirements](https://ai.google.dev/gemini-api/docs/grounding/search-suggestions).

The data you need to meet these display requirements, and to build e.g. footnote functionality will be saved to the response's `additionalContent` property.

```php
// The Google supplied and styled widget to click through to results.
$response->additionalContent['searchEntryPoint'];

// The search queries made by the model
$response->additionalContent['searchQueries'];

// The citations data is available as an array of MessagePartWithCitations
$response->additionalContent['citations'];
```

`citations` is an array of `MessagePartWithCitations`, which you can use to build up footnotes as follows:

```php
use Prism\Prism\ValueObjects\MessagePartWithCitations;
use Prism\Prism\ValueObjects\Citation;

$text = '';
$footnotes = [];

$footnoteId = 1;

/** @var MessagePartWithCitations $part */
foreach ($response->additionalContent['citations'] as $part) {
    $text .= $part->outputText;
    
    /** @var Citation $citation */
    foreach ($part->citations as $citation) {
        $footnotes[] = [
            'id' => $footnoteId,
            'title' => $citation->sourceTitle,
            'uri' => $citation->source,
        ];

        $text .= '<sup><a href="#footnote-'.$footnoteId.'">'.$footnoteId.'</a></sup>';

        $footnoteId++;
    }
}

// Pass $text and $footnotes to your frontend.
```

## Structured Output

Gemini supports structured output, allowing you to define schemas that constrain the model's responses to match your exact data structure requirements.

```php
use Prism\Prism\Facades\Prism;
use Prism\Prism\Enums\Provider;
use Prism\Prism\Schema\ObjectSchema;
use Prism\Prism\Schema\StringSchema;

$schema = new ObjectSchema(
    name: 'movie_review',
    description: 'A structured movie review',
    properties: [
        new StringSchema('title', 'The movie title'),
        new StringSchema('rating', 'Rating out of 5 stars'),
        new StringSchema('summary', 'Brief review summary'),
    ],
    requiredFields: ['title', 'rating', 'summary']
);

$response = Prism::structured()
    ->using(Provider::Gemini, 'gemini-2.0-flash')
    ->withSchema($schema)
    ->withPrompt('Review the movie Inception')
    ->asStructured();

// Access structured data
dump($response->structured);
```

### Flexible Types with anyOf

For fields that can match multiple types or structures, use `AnyOfSchema`. This is useful for polymorphic data or when a field might contain different shapes:

```php
use Prism\Prism\Facades\Prism;
use Prism\Prism\Enums\Provider;
use Prism\Prism\Schema\AnyOfSchema;
use Prism\Prism\Schema\ObjectSchema;
use Prism\Prism\Schema\StringSchema;
use Prism\Prism\Schema\NumberSchema;

// Simple example: value can be string or number
$schema = new ObjectSchema(
    'response',
    'API response with flexible value',
    [
        new AnyOfSchema(
            schemas: [
                new StringSchema('text', 'Text value'),
                new NumberSchema('number', 'Numeric value'),
            ],
            name: 'value',
            description: 'Can be either text or number'
        ),
    ],
    ['value']
);

$response = Prism::structured()
    ->using(Provider::Gemini, 'gemini-2.5-flash')
    ->withSchema($schema)
    ->withPrompt('Extract the value from: "The answer is 42"')
    ->asStructured();

// $response->structured['value'] could be "42" (string) or 42 (number)
```

For complex polymorphic structures, `anyOf` can distinguish between entirely different object types:

```php
use Prism\Prism\Facades\Prism;
use Prism\Prism\Enums\Provider;
use Prism\Prism\Schema\AnyOfSchema;
use Prism\Prism\Schema\ObjectSchema;
use Prism\Prism\Schema\StringSchema;
use Prism\Prism\Schema\NumberSchema;

$articleSchema = new ObjectSchema(
    'article',
    'A blog article',
    [
        new StringSchema('title', 'Article title'),
        new StringSchema('content', 'Full article text'),
        new StringSchema('author', 'Author name'),
    ],
    ['title', 'content']
);

$imageSchema = new ObjectSchema(
    'image',
    'An image post',
    [
        new StringSchema('url', 'Image URL'),
        new StringSchema('caption', 'Image caption'),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prism-php/prism](https://github.com/prism-php/prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
