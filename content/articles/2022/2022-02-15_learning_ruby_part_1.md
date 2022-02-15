Title: Learning Ruby Like it's 2009 — Part 1
Tags: learning, ruby, bundler, learning-ruby-like-its-2009
Language: en
Summary: First part of my journey to learn ruby and rails. Focusing on bundler and version management.

As announced in my previous post I will document my journey of learning Ruby
and especially Ruby on Rails. Or how a friend of mine put it after hearing
about my idea:

> Blogging about Rails like it's 2009 😄.
>
> [@dbrgn](https://blog.dbrgn.ch/about/) February, 2022

## Learning Ruby like it's 2009

One of my first exercises was to investigate a case where an automatic update
done by the [depfu](https://depfu.com/)-bot failed the tests and as such
couldn't be merged. It looked similar to
[this](https://github.com/renuo/pingen-client/pull/15) but failed the tests and
had a lot more updates including some major ones.

Already knowing a few programming languages and package managers my first idea
was: «Easy: Let's just first do all the minor and patch upgrades and see if
this breaks anything». Assuming the gems follow semantic versioning this should
just work.

## Bundler and version specifier

First thing I noticed is that Bundler, while recommending semantic versioning,
doesn't directly support semantic version specifiers like for example
[cargo](https://doc.rust-lang.org/cargo/reference/semver.html) does. It does
support defining your own ranges like the following:
```ruby
gem 'foo', '>= 2.2.0', '< 3.0'`
```
or using the `~>` operator which achives the same thing:
```ruby
gem 'foo', '~> 2.2'
```

For
[cargo](https://doc.rust-lang.org/cargo/reference/resolver.html#semver-compatibility)
one would just specify
```toml
foo = "2.2.0"
```
with the same result and in
[npm](https://docs.npmjs.com/about-semantic-versioning) we can use the `^`
operator:
```json
"foo": "^2.2.0"
```
For bundler there is an [issue on
GitHub](https://github.com/rubygems/rubygems/issues/1919) to add a SemVer
operator since 2017.

The code base I was working on didn't have a lot of version specifiers. The
reason is that Renuo generally tries to use the latest versions of all Ruby
dependencies for applications. This is possible since we have extensive test
suites which we trust and it enables an open and fast upgrade path for
customers if they need new features in the future.

Exceptions are made if a gem update causes problems or if it is a very central
gem like rails. To still have reproducible environments for applications the
`Gemfile.lock` file is of course checked into source control as it is [best
practice](https://yehudakatz.com/2010/12/16/clarifying-the-roles-of-the-gemspec-and-gemfile/).

So just running `bundle update` would update to the latest major version of
almost all dependencies, which was exactly the thing I didn't want to do.

Luckily bundler has the
[`--minor`](https://bundler.io/v2.3/man/bundle-update.1.html) option which one
can pass to the `update` command which should *Prefer updating only to next
minor version.*

## A minor inconvenience

So here we go executing `bundler update --minor`! And the result was...
Nothing! It just didn't update anything!

Confused I started reading up documentation, version specifiers, tried to
update individual packages with `bundle update --minor $package` (which worked)
and finally found <https://github.com/rubygems/rubygems/issues/3360>. Apparently
it's a known issue since 2018 that bundle update just fails to do the `--patch`
and `--minor` updates.

From the same GitHub issue I copied the handy shell one-liner `bundle update
$(bundle list | awk '$1 ~ /^\*/ {print $2}' | grep -v bundler) --minor` which
did the job!

Doing just the semver compatible updates worked, all the tests did pass and we
could deploy it. My shattered trust in the Ruby ecosystem from the `bunlder`
failure was restored!

All in all bundler is a decent package manager and I love that the Ruby
ecosystem seems to embrace semantic versioning and one can do fearless upgrades
to get bug- and security fixes of the dependencies.
