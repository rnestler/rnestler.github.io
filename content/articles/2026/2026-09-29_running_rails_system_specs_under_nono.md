Title: Getting Rails System Specs to Run Under nono
Tags: nono, Rails, Testing, Selenium, Sandbox, Ruby
Language: en
Status: draft
Summary: What breaks when you run Capybara system specs inside nono's sandbox, and how to fix it.

When I use agentic AI development harnesses like [Claude Code] or [OpenCode] I
always sandbox them using [nono] to be comfortable to let them work on their
own.

While this works great most of the time and it's quite straight forward to
create new profile to give the agent the correct permissions:

```json
{
  "meta": {
    "name": "rails",
    "version": "0.1.0"
  },
  "filesystem": {
    "allow": [
      "$HOME/.cache/rubocop_cache"
    ],
    "read": [
      "$HOME/.local/share/mise/installs/ruby",
      "$HOME/.local/share/mise/installs/node/",
      "$HOME/.local/share/ruby-advisory-db",
      "$HOME/.cache/Cypress",
      "$HOME/.config/yarn"
    ],
    "read_file": [
      "/etc/passwd",
      "~/.yarnrc"
    ],
    "unix_socket": [
      "/var/run/postgresql/.s.PGSQL.5432"
    ]
  },
  "environment": {
    "set_vars": {
      "BUNDLE_USER_CONFIG": "/dev/null"
    }
  },
  "workdir": {
    "access": "readwrite"
  }
}
```

I just added permissions whenever the agent ran into issues. But for system
specs, which need to run a (headless) browser it was quite tough to get it
working.

[Claude Code]: https://code.claude.com/docs/
[OpenCode]: https://opencode.ai/
[nono]: https://nono.sh/

## The Problem

While the agent can work very well already with just running the model and
request specs, every now and then it would be useful if it could run the
end-to-end / system specs by itself as well.

The main problem is, that browser require quite some access to the system and
thus are very difficult to sandbox. The initial error when trying to execute the system specs looked like this:

```
$ NO_COVERAGE=true nono run --allow-cwd --profile rails --extends mise bundle exec rspec ./spec/system/logins/profile_spec.rb:16

Profile
2026-09-29 09:05:53 INFO Selenium [:logger_info] Details on how to use and modify Selenium logger:
  https://selenium.dev/documentation/webdriver/troubleshooting/logging

2026-09-29 09:05:53 INFO Selenium [:selenium_manager] No exit status for: ["/home/raphael/.local/share/mise/installs/ruby/3.4.10/lib/ruby/gems/3.4.0/gems/selenium-webdriver-4.46.0/bin/linux/selenium-manager", "--browser", "chrome", "--language-binding", "ruby", "--output", "json"]. Assuming success if result is present.
2026-09-29 09:05:53 ERROR Selenium Exception occurred: no implicit conversion of nil into String
2026-09-29 09:05:53 ERROR Selenium Backtrace:
	/home/raphael/.local/share/mise/installs/ruby/3.4.10/lib/ruby/gems/3.4.0/gems/selenium-webdriver-4.46.0/lib/selenium/webdriver/common/platform.rb:134:in 'File.file?'

.... Huge wall of backtrace

     1.1) Failure/Error: page.driver.browser.manage.window.resize_to(1920, 1080)

          Selenium::WebDriver::Error::NoSuchDriverError:
            Unable to obtain chromedriver; For documentation on this error, please visit: https://www.selenium.dev/documentation/webdriver/troubleshooting/errors/driver_location

...

Finished in 0.40964 seconds (files took 1.63 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/system/logins/profile_spec.rb:16 # Profile can update profile information
```

## Investigating

Since it's completely not obvious what happens and where it fails, let's try to
debug it by running the command that failed:

```
2026-09-29 09:05:53 INFO Selenium [:selenium_manager] No exit status for: ["/home/raphael/.local/share/mise/installs/ruby/3.4.10/lib/ruby/gems/3.4.0/gems/selenium-webdriver-4.46.0/bin/linux/selenium-manager", "--browser", "chrome", "--language-binding", "ruby", "--output", "json"]. Assuming success if result is present.
```

Running it outside the sandbox gives us

```
$ /home/raphael/.local/share/mise/installs/ruby/3.4.10/lib/ruby/gems/3.4.0/gems/selenium-webdriver-4.46.0/bin/linux/selenium-manager "--browser" "chrome" "--language-binding" "ruby" "--output" "json"
{
  "logs": [
    {
      "level": "INFO",
      "timestamp": 1790680352,
      "message": "Driver path: /usr/bin/chromedriver"
    },
    {
      "level": "INFO",
      "timestamp": 1790680352,
      "message": "Browser path: /usr/bin/chromium"
    }
  ],
  "result": {
    "code": 0,
    "message": "",
    "driver_path": "/usr/bin/chromedriver",
    "browser_path": "/usr/bin/chromium"
  }
}
```

So it just tries to find the binaries to run the browser. Let's execute it in the sandbox:
```
nono run --allow-cwd --profile rails /home/raphael/.local/share/mise/installs/ruby/3.4.10/lib/ruby/gems/3.4.0/gems/selenium-webdriver-4.46.0/bin/linux/selenium-manager "--browser" "chrome" "--language-binding" "ruby" "--output" "json"

  nono v0.78.0
  Capabilities:
  ────────────────────────────────────────────────────
   r+w  /home/raphael/.cache/rubocop_cache (dir)
    r   /home/raphael/.local/share/mise/installs/ruby (dir)
    r   /home/raphael/.local/share/mise/installs/node (dir)
    r   /home/raphael/.local/share/ruby-advisory-db (dir)
    r   /home/raphael/.cache/Cypress (dir)
    r   /home/raphael/.config/yarn (dir)
    r   /etc/passwd (file)
    r   /home/raphael/.yarnrc (file)
    r   /run/postgresql/.s.PGSQL.5432 (file)
   r+w  /home/raphael/projects/renuo/availabill (dir)
       + 49 system/group paths (-v to show)
  sock  /run/postgresql/.s.PGSQL.5432
   net  outbound allowed
  ────────────────────────────────────────────────────

  Applying sandbox...


thread 'main' (1207818) panicked at src/metadata.rs:102:60:
called `Result::unwrap()` on an `Err` value: Os { code: 13, kind: PermissionDenied, message: "Permission denied" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
[nono] Session stopped.

Command killed by signal 6 / SIGABRT (exit code 134).

No path denials were observed during this session.
The failure may be unrelated to sandbox restrictions.
```

This just shows some executable being unable to access something. Not very
helpful. Running it with `RUST_BACKTRACE=full` also didn't reveal anything,
probably because the binary doesn't contain debug symbols. To investigate
further I tried to install selenium-manager manually.

I found it on the [AUR](https://aur.archlinux.org/packages/selenium-manager),
and enabled debug symbols:

```diff
diff --git a/PKGBUILD b/PKGBUILD
index a861aeb..11e1958 100644
--- a/PKGBUILD
+++ b/PKGBUILD
@@ -14,7 +14,7 @@ makedepends=(cargo python)
 checkdepends=()
 source=("https://github.com/SeleniumHQ/${_name}/archive/refs/tags/${_name}-${_pkgver}.tar.gz")
 sha256sums=('bd710afb49760e6d5c34dcc16a8f63099e3fc866950c56b9d3bee16b309453a3')
-options=('!lto')
+options=('!lto' '!strip')

 prepare() {
   cd "${_name}-${_name}-${_pkgver}/rust"
@@ -26,12 +26,12 @@ build() {
   cd "${_name}-${_name}-${_pkgver}/rust"
   export RUSTUP_TOOLCHAIN=stable
   export CARGO_TARGET_DIR=target
-  cargo build --frozen --release --all-features
+  cargo build --frozen --all-features
 }

 package() {
   cd "${_name}-${_name}-${_pkgver}/rust"
-  install -Dm0755 -t "$pkgdir/usr/bin/" "target/release/$pkgname"
+  install -Dm0755 -t "$pkgdir/usr/bin/" "target/debug/$pkgname"
   local site_packages=$(python -c "import site; print(site.getsitepackages()[0])")
   install -d "$pkgdir/$site_packages/selenium/webdriver/common/linux"
   ln -sf "/usr/bin/$pkgname" "$pkgdir/$site_packages/selenium/webdriver/common/linux"
```

Running it showed where the issue is:
```
$ RUST_BACKTRACE=1 nono run --allow-cwd --profile rails selenium-manager "--browser" "chrome" "--language-binding" "ruby" "--output" "json"
...
thread 'main' (1259896) panicked at src/metadata.rs:102:60:
called `Result::unwrap()` on an `Err` value: Os { code: 13, kind: PermissionDenied, message: "Permission denied" }
stack backtrace:
   0: __rustc::rust_begin_unwind
             at /rustc/48a229ceaefd4985c50990b14116b6d856af0985/library/std/src/panicking.rs:679:5
   1: core::panicking::panic_fmt
             at /rustc/48a229ceaefd4985c50990b14116b6d856af0985/library/core/src/panicking.rs:80:14
   2: core::result::unwrap_failed
             at /rustc/48a229ceaefd4985c50990b14116b6d856af0985/library/core/src/result.rs:1870:5
   3: <core::result::Result<std::fs::File, core::io::error::Error>>::unwrap
             at /home/raphael/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/result.rs:1231:23
   4: selenium_manager::metadata::get_metadata
             at ./src/selenium-selenium-4.48.0/rust/src/metadata.rs:102:60
   5: selenium_manager::prune_old_cache_entries
             at ./src/selenium-selenium-4.48.0/rust/src/lib.rs:1781:24
   6: selenium_manager::main
             at ./src/selenium-selenium-4.48.0/rust/src/main.rs:286:5
   7: <fn() as core::ops::function::FnOnce<()>>::call_once
             at /home/raphael/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

Looking at [the
source](https://github.com/SeleniumHQ/selenium/blob/selenium-4.48.0/rust/src/metadata.rs#L99)
I noticed that it should show a trace log message indicating which file fails
to read:
```rust
pub fn get_metadata(log: &Logger, cache_path: &Option<PathBuf>) -> Metadata {
    if let Some(cache) = cache_path {
        let metadata_path = get_metadata_path(cache.clone());
        log.trace(format!("Reading metadata from {}", metadata_path.display()));

        if metadata_path.exists() {
            let metadata_file = File::open(&metadata_path).unwrap();
...
```

Also it shows once again, that a bare `.unrwap()` is just bad style in Rust,
since it completely hides errors from the user. I opened a PR to fix this [^1].

So creating a custom selenium profile with the following content fixes running
selenium-manager in nono:

```json
{
  "meta": {
    "name": "selenium"
  },
  "filesystem": {
    "allow": [
      "$HOME/.cache/selenium/"
    ]
  }
}
```

[^1]: https://github.com/SeleniumHQ/selenium/pull/18087

### Shared memory

Chrome wants `/dev/shm`. nono restricts that. Giving it access to `/dev/shm`
led to the process hanging. Just telling it not to use it fixed it.


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
