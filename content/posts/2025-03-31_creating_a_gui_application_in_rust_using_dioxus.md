+++
title = "Creating a GUI Application in Rust using Dioxus"
draft = true

[taxonomies]
tags = ["Coding", "Rust", "Dioxus"]
+++

My [previous post about Vibe Coding](@/posts/2025-03-28_vibe_coding.md) got me thinking whether, perhaps, it is hard to create a GUI application in Rust. I have primarily been a backend developer with a little bit of web frontend experience using [Vue.js](https://vuejs.org/).

[Are We GUI Yet](https://areweguiyet.com/#ecosystem) lists the various Rust GUI frameworks. The description for [Dioxus](https://dioxuslabs.com/) sounded intriguing.Hopefully this is not drinking the Kool-Aid. See [Google Web Toolkit](https://www.gwtproject.org/), an attempt to allow writing frontend code in Java which would then be transpiled into Javascript. I had to work with a legacy app using that for its frontend and it was a disaster. THe current version might be better. We were using an old version that had gone out of support over 10 years ago. Fun times...

I am going to be reading through the [Dioxus Getting Started Guide](https://dioxuslabs.com/learn/0.6/getting_started/) and learning as I write this. This is not a tutorial. It is my journey learning Dioxus and attempting a GUI application.

<!-- more -->

# Getting the tooling all set up

* *Rust* ✅ - already installed
* *Dioxus CLI* ✅ - had to get the OpenSSL dependency set up as documented in the link in the getting started docs

# Building the *HotDog* application
## [Setting Up Tooling](https://dioxuslabs.com/learn/0.6/guide/tooling)
The getting started documentation is going to have us build Tinder for Dogs. It says the `wasm32-unknown-unknown` toolchain must be installed. I do not think I have that installed. I think toolchain is not the right word. I think that is technically a compilation target.

```bash
rustup target add wasm32-unknown-unknown
```

Turns out I already have it installed. I [sent a pull request to the project](https://github.com/DioxusLabs/docsite/pull/472), so hopefully that is fixed by the time you read it.

## [Creating a new app](https://dioxuslabs.com/learn/0.6/guide/new_app)
I run `dx new hot_dog` as instructed and choose the options specified in the documentation. The tooling is very slick.

```bash
❯ dx new hot_dog
✔ 🤷   Which sub-template should be expanded? · Bare-Bones
✔ 🤷   Do you want to use Dioxus Fullstack? · false
✔ 🤷   Do you want to use Dioxus Router? · false
✔ 🤷   Do you want to use Tailwind CSS? · false
✔ 🤷   Which platform do you want DX to serve by default? · Web
  71.834s  INFO Generated project at /Users/brianbustin/projects/hot_dog

`cd` to your project and run `dx serve` to start developing.
If using Tailwind, make sure to run the Tailwind CLI.
More information is available in the generated `README.md`.

Build cool things! ✌️
```

I really like how it documents the next steps. This makes using it more approachable. Even if you aren't following the getting started and making Dog Tinder.

Then I run `dx serve` as instructed, open http://127.0.0.1:8080 in my browser, and there is a Dioxus website. Very nice so far. I am curious what code leveraging the library is going to be like.

The getting started documentation is well done. It is going over the project structure and explaining everything with just the right level of detail. Hats off to the Dioxus team. Nicely done.

I clear away all of the boilerplate, as instructed. The webpage in my beowser hot reloads and it says `HotDog!` which is exactly what I am thinking as this is going so smoothly.

## [First Component](https://dioxuslabs.com/learn/0.6/guide/component)

> In Dioxus, apps are comprised of individual functions called Components that take in some Properties and render an Element

Makes sense to me.

The starting point for `src/main.rs` is:

```rust
use dioxus::prelude::*;

fn main() {
    dioxus::launch(App);
}

#[component]
fn App() -> Element {
    rsx! {
        "HotDog!"
    }
}
```

No I add a component called `DogApp` which accepts a `breed` parameter and then I use it in the `App function.

```rust
use dioxus::prelude::*;

fn main() {
    dioxus::launch(App);
}

#[component]
fn App() -> Element {
    rsx! {
        DogApp { breed: "Brittany Spaniel" }
    }
}

#[component]
fn DogApp(breed: String) -> Element {
    rsx! {
        "Breed: {breed}"
    }
}
```

The webpage displays `Breed: Brittany Spaniel`. I have created a component. The framework does some cool work behind the scenes to make this work. The documents describe that a Struct called `DogAppProps` (name of the component with `Props` at the end). If you are building an API, the docs recommend creating those structs manually.

## [Creating UI with RSX](https://dioxuslabs.com/learn/0.6/guide/rsx)
> ..all quoted strings in RSX imply format!() automatically, so you can define a variable outside your markup and use it in your strings without an explicit

I like that. It makes things a little less verbose.

One thing I do not like is that if the build fails, the how reloading stops. Even thouch `dx serve` is still running. To get around this, I need to press `ctrl + c` and then run it again. Not a big deal, just a minor annoyance.

Now the code looks like this:

```rust
use dioxus::prelude::*;

fn main() {
    dioxus::launch(App);
}

#[component]
fn App() -> Element {
    rsx! {
        div { id: "title",
            h1 { "HotDog! 🌭"  }
        }
        div { id: "dogview",
            img { src: "https://www.selectadogbreed.com/media/1550/brittanyspaniel_puppy.jpg" }
        }
        div { id: "buttons",
            button { id: "skip", "skip" }
            button { id: "save", "save!" }
        }
    }
}
```

## [Styling and Assets](https://dioxuslabs.com/learn/0.6/guide/assets)

Styling in Dioxus uses CSS. So you essentially do this...

```rust
use dioxus::prelude::*;

static CSS: Asset = asset!("/assets/main.css");

fn main() {
    dioxus::launch(App);
}

#[component]
fn App() -> Element {
    rsx! {
        document::Stylesheet { href: CSS }
    }
}
```

I make the changes above (but leave in all of the other code). Then I copy in the stylesheet from the tutorial into `/assets/main.css`. It looks nice and the hot reloading just works. Pretty sweet!

## [Adding State](https://dioxuslabs.com/learn/0.6/guide/state)

Now we:

* Split the Title and DogView out of App since DogView is going to have associated state and title will not.
* Add event handlers to DogView.
* Make the image source part of the DogView state and define a starting image with the `use_signal` function. (Note to future self: Read the [State Management Guide](https://dioxuslabs.com/learn/0.6/essentials/state/#))

```rust
use dioxus::prelude::*;

static CSS: Asset = asset!("/assets/main.css");

fn main() {
    dioxus::launch(App);
}

#[component]
fn App() -> Element {
    rsx! {
        document::Stylesheet { href: CSS }
        Title {}
        DogView {}
    }
}

#[component]
fn Title() -> Element {
    rsx! {
        div { id: "title",
            h1 { "HotDog! 🌭" }
        }
    }
}

#[component]
fn DogView() -> Element {
    let skip = move |evt| {};
    let save = move |evt| {};

    let img_src = use_signal(|| "https://www.selectadogbreed.com/media/1550/brittanyspaniel_puppy.jpg");

    rsx! {
        div { id: "dogview",
            img { src: "{img_src}" }
        }
        div { id: "buttons",
            button { onclick: skip, id: "skip", "skip" }
            button { onclick: save, id: "save", "save!" }
        }
    }
}
```

## [Fetching Data](https://dioxuslabs.com/learn/0.6/guide/data_fetching)
Dog Tinder needs to be able to get other dogs for when you skip. This app is going to use the [dog.ceo/dog-api](https://dog.ceo/dog-api/) to get images. So we need a library to call the API, and also a library to deserialize the response.

```bash
cargo add reqwest --features json
cargo add serde --features derive
```

It looks like Dioxus can fetch the image on its own. The tricky part is calling the dog.ceo API and getting the URL to the image to display.

The code looks like this now.

```rust
use dioxus::prelude::*;

static CSS: Asset = asset!("/assets/main.css");

fn main() {
    dioxus::launch(App);
}

#[component]
fn App() -> Element {
    rsx! {
        document::Stylesheet { href: CSS }
        Title {}
        DogView {}
    }
}

#[component]
fn Title() -> Element {
    rsx! {
        div { id: "title",
            h1 { "HotDog! 🌭" }
        }
    }
}

#[component]
fn DogView() -> Element {
    let mut img_src = use_resource(|| async move {
        reqwest::get("https://dog.ceo/api/breeds/image/random")
            .await
            .unwrap()
            .json::<DogApi>()
            .await
            .unwrap()
            .message
    });

    rsx! {
        div { id: "dogview",
            img { src: img_src.cloned().unwrap_or_default() }
        }
        div { id: "buttons",
            button { onclick: move |_| img_src.restart(), id: "skip", "skip" }
            button { onclick: move |_| img_src.restart(), id: "save", "save!" }
        }
    }
}

#[derive(serde::Deserialize)]
struct DogApi {
    message: String,
}
```

## [Adding a Backend](https://dioxuslabs.com/learn/0.6/guide/backend)
It looks like the dx command can do this stuff for you automatically if you chose full stack to begin with.

Edit the `cargo.toml` file and:
* add the `fullstack` feature to the `dioxus` dependency
* remove the `default = ["web"]` feature
* add the `server = ["dioxus/server"]` feature

My `cargo.toml` now looks like this.

```toml
[package]
name = "hot_dog"
version = "0.1.0"
authors = ["Brian Bustin <>"]
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
dioxus = { version = "0.6.0", features = ["fullstack"] }
reqwest = { version = "0.12.15", features = ["json"] }
serde = { version = "1.0.219", features = ["derive"] }

[features]
web = ["dioxus/web"]
desktop = ["dioxus/desktop"]
mobile = ["dioxus/mobile"]
server = ["dioxus/server"]

[profile]

[profile.wasm-dev]
inherits = "dev"
opt-level = 1

[profile.server-dev]
inherits = "dev"

[profile.android-dev]
inherits = "dev"
```

Then `ctrl+c` `dx serve` and instead run `dx serve --platform web` since it no longer has a default.

To be continued...
