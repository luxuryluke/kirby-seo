# Sitemap

Sitemaps are enabled by default. You can disable them by setting `tobimori.seo.sitemap.active` to `false` in your `config.php`.

![Sitemap Example](/docs/_assets/sitemap-example.png)

A nice looking stylesheet inspired by the Kirby panel will be added automatically as well.

## [Options](/config/options.php)

Some options are available to customize the default behaviour of the sitemap generator.

| Option                     | Default                                    | Accepts            | Description                                                                          |
| -------------------------- | ------------------------------------------ | ------------------ | ------------------------------------------------------------------------------------ |
| `sitemap.active`           | `true`                                     | Boolean            | Enable/disable the entire sitemap module                                             |
| `sitemap.lang`             | `en`                                       | String / Closure   | Language of the sitemap (human readable strings)                                     |
| `sitemap.changefreq`       | `weekly`                                   | String / Closure   | Change frequency of an URL                                                           |
| `sitemap.priority`         | [options.php](/config/options.php#L69)     | String / Closure   | Priority of an URL                                                                   |
| `sitemap.groupByTemplate`  | `false`                                    | Boolean            | Create separate sitemaps for each template type                                      |
| `sitemap.excludeTemplates` | `['error']`                                | Array with strings | Exclude templates from default sitemap generator                                     |
| `sitemap.generator`        | [sitemap.php](/config/options/sitemap.php) | Closure            | Generator function for the sitemap, [see below](#customizing-the-generator-function) |

Options allow you to fine tune the behaviour of the plugin. You can set them in your `config.php` file:

```php
return [
    'tobimori.seo.sitemap' => [
        'groupByTemplate' => false, // Create separate sitemaps for each template type
        'excludeTemplates' => ['error'], // Exclude templates from sitemap
        'changefreq' => 'weekly', // Change frequency, can be a string or a function
        'priority' => fn (Page $p) => number_format(($p->isHomePage()) ? 1 : max(1 - 0.2 * $p->depth(), 0.2), 1), // Priority, can be a string or a function
    ]
];
```

## Customizing the generator function

Sitemaps are generated by a generator function, which you can override in your `config.php` file, if the above mentioned config options don't fulfill your needs. The default function looks something like this:

The generator function allows you to programmatically generate sitemaps and add URLs to them. When a sitemap route is requested, this generator function will automatically be invoked to add content to the sitemap.

```php
return [
    `tobimori.seo.sitemap.generator` => function (SitemapIndex $sitemap) {
        $pages = site()->index()->filter(fn ($page) => $page->metadata()->robotsIndex()->toBool())->group('intendedTemplate');

        foreach ($pages as $group) {
            $index = $sitemap->create($group->first()->intendedTemplate()->name());

            foreach ($group as $page) {
            $url = $index->createUrl($page->metadata()->canonicalUrl())
                ->lastmod($page->modified())
                ->changefreq(option('tobimori.seo.sitemap.changefreq')($page))
                ->priority(option('tobimori.seo.sitemap.priority')($page));
            }
        };
    };
];
```

(This is a simplified version of the actual function, which you can find in the [source code](/config/options/sitemap.php).)

If only a single sitemap is defined, the sitemap index route will directly render that sitemap instead of first generating a sitemap index file.
Note that if you provide a custom sitemap generator function, the standard sitemap configuration options are ignored. However, you can access and utilize those configuration values within your own generator function if desired.