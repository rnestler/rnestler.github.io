Title: Getting Rails System Specs to Run Under nono
Tags: nono, Rails, Testing, Selenium, Sandbox, Ruby
Language: en
Status: draft
Summary: What breaks when you run Capybara system specs inside nono's sandbox, and how to fix it.

When I use agentic AI development harnesses like [Claude Code] or [OpenCode] I
always sandbox them using [nono] to be comfortable to let them work on their
own.

While this works great most of the time and it's quite straight forward to
create new profile to give the agent the correct permissions. But for system
specs, which need to run a (headless) browser it was quite tough to get it
working.

[Claude Code]: https://code.claude.com/docs/
[OpenCode]: https://opencode.ai/
[nono]: https://nono.sh/

## The Problem

Running `rspec spec/system` directly: fine.  
Running `nono run -- rspec spec/system`: fails.

The error looks something like:

```
# TODO: insert actual error
```

Not immediately obvious what nono blocked.

## Investigating

### Browser binary

Chromium installs to some system path. Is it inside the sandbox?

```bash
which chromium
```

### Shared memory

Chrome wants `/dev/shm`. nono restricts that.

### Child processes

Selenium starts chromedriver, which starts Chromium. Does the restriction inherit?

## The Fix

`rails_helper.rb` or `spec/support/capybara.rb`:

```ruby
Capybara.register_driver :nono_chrome do |app|
  options = Selenium::WebDriver::Chrome::Options.new
  options.add_argument("--headless=new")
  options.add_argument("--disable-dev-shm-usage")
  # TODO: any other flags?

  Capybara::Selenium::Driver.new(app, browser: :chrome, options: options)
end

Capybara.javascript_driver = :nono_chrome
```

And the nono invocation:

```bash
nono run --allow /usr/bin/chromium --allow /dev/shm -- rspec spec/system
```

Or a profile:

```json
{
  "extends": "opencode",
  "paths": {
    "/usr/bin/chromium": "read",
    "/dev/shm": "readwrite"
  }
}
```

## Verification

Run the suite. It passes. Attach a screenshot.

## What still breaks?

Downloads? Screenshots to a restricted path? Needs testing.
