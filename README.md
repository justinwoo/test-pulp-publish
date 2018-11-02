# Pursuit docs publish notes

See <https://github.com/purescript/pursuit/blob/master/static/help-docs/authors.md> for the other details

Let's look at the output:

## Resolutions

We need resolutions to work with (saved to parent path because purs publish bitches about git state).

```
bower list --json --offline > ../resolutions.json
```

Now for simple-json, the result is already 21MB, so let's look at some properties of this shit instead in an empty project:

```
> jq '.dependencies = "..."' ../resolutions.json
{
  "endpoint": {
    "name": "test-pulp-publish",
    "source": "/home/justin/Code/test-pulp-publish",
    "target": "*"
  },
  "canonicalDir": "/home/justin/Code/test-pulp-publish",
  "pkgMeta": {
    "name": "test-pulp-publish",
    "ignore": [
      "**/.*",
      "node_modules",
      "bower_components",
      "output"
    ],
    "dependencies": {
      "purescript-prelude": "^4.1.0",
      "purescript-console": "^4.1.0",
      "purescript-effect": "^2.0.0"
    },
    "devDependencies": {
      "purescript-psci-support": "^4.0.0"
    }
  },
  "dependencies": "...",
  "nrDependants": 0
}
```

Each of the dependencies is its own nested structure of the whole. See:

```
> jq '.dependencies."purescript-console"
> | .dependencies = (.dependencies | keys | map("...definition of " + .))
> | .endpoint = "...same structure as overall schema"
> | .pkgMeta = "...same structure as overall schema"
> ' ../resolutions.json
{
  "endpoint": "...same structure as overall schema",
  "canonicalDir": "/home/justin/Code/test-pulp-publish/bower_components/purescript-console",
  "pkgMeta": "...same structure as overall schema",
  "dependencies": [
    "...definition of purescript-effect",
    "...definition of purescript-prelude"
  ],
  "nrDependants": 1
}
```

So you can see why Bower's naive implementation stack overflows so easily, since previous usages of Bower have never been so deeply nested with so many dependencies, as it was created to mostly handle asset dependencies which do not branch largely.

Now we can prepare the publishing payload using `purs publish`. We have to take care of annoyances with the bower.json checker, but after that and making a dummy git tag, we can publish:

```
purs publish --manifest bower.json --resolutions ../resolutions.json > ../publish_result.json
```

Fin
