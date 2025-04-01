+++
title = "Creating a GUI Application in Rust using Dioxus"

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

Each of the headers in this section link to the corresponding page in the [Dioxus documentation](https://dioxuslabs.com/learn/0.6/).

## [Setting Up Tooling](https://dioxuslabs.com/learn/0.6/guide/tooling)
The getting started documentation is going to have us build Tinder for Dogs. It says the `wasm32-unknown-unknown` toolchain must be installed. I do not think I have that installed. Toolchain is not the right word, it technically is a compilation target.

I [sent a pull request to the project](https://github.com/DioxusLabs/docsite/pull/472) to fix this tiny issue and it was quickly accepted.

```bash
rustup target add wasm32-unknown-unknown
```

Turns out I already have it installed.

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

I clear away all of the boilerplate, as instructed. The webpage in my browser hot reloads and it says `HotDog!` which is exactly what I am thinking as this is going so smoothly.

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

One thing I do not like is that if the build fails, the hot reloading sometimes stops even though `dx serve` is still running. To get around this, I need to press `ctrl + c` and then run it again. Not a big deal, just a minor annoyance.

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

It looks like Dioxus can fetch the image on its own. The tricky part is calling the dog.ceo API and getting the URL for the image to display.

The code looks like this now:

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
It looks like the dx command does these steps for you automatically if you choose full stack when creating your project.

Edit the `cargo.toml` file and:
* add the `fullstack` feature to the `dioxus` dependency
* remove the `default = ["web"]` feature
* add the `server = ["dioxus/server"]` feature

My `cargo.toml` now looks like this.

```toml
[package]
name = "hot_dog"
version = "0.1.0"
authors = ["Brian Bustin <_@_._>"]
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

Then we create a server endpoint to save the dog which takes an image parameter. It just saves it to a text file. Then the client code needs to call the server function. I just followed the steps.

The documentation does mention that you need to be careful making sure any server-side secrets are not exposed to the client. In that case, you can put the server code in a different module behind `#[cfg(feature = "server")]`. Hopefully they will get more into that later on. The code below does not have any secrets and does not split the code. Everything is also in `main.rs`. In a real project, some more structure would help keep things neat.

The other thing I would want to improve over the tutorial code are all the `unwraps`. It might be good to have more explicit error handling. This is just a tutorial app though. 😁

It works though!

Here is how the code looks...
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
            button { id: "skip", onclick: move |_| img_src.restart(), "skip" }
            button {
                id: "save",
                onclick: move |_| async move {
                    let current = img_src.cloned().unwrap();
                    img_src.restart();
                    _ = save_dog(current).await;
            },
            "save!" }
        }
    }
}

#[derive(serde::Deserialize)]
struct DogApi {
    message: String,
}

#[server]
async fn save_dog(image: String) -> Result<(), ServerFnError> {
    use std::io::Write; // keeping server-specific imports out of the client code

    // Append to `dogs.txt`. If it does not exist, then create it.
    let mut file = std::fs::OpenOptions::new()
        .write(true)
        .append(true)
        .create(true)
        .open("dogs.txt")
        .unwrap();

    // Write the image URL to the file and then insert a new line
    file.write_fmt(format_args!("{image}\n"));

    Ok(())
}
```

## [Working with Databases](https://dioxuslabs.com/learn/0.6/guide/databases)

We are going to get the app working with a SQLite database.

I chose to use the cargo command to add the dependency to the server feature rather than editing `cargo.toml` by hand. This may come to bite me as the resulting `cargo.toml` is different from the example. (Spoiler: It does... do not run this command or you'll have to manually clean up your `cargo.toml`.)

```bash
cargo add rusqlite --target server
```

Then I add the database connection code as per the tutorial.

The way I chose to add the `rusqlite` dependency did come to bite me. IT turns out I had confused a `target` with a `feature`. I undid the changes in my `cargo.toml` and made it reflect the how it is in the tutorial, but I used the latest version.

I use [DB Browser for SQLite (DB4S)](https://github.com/sqlitebrowser/sqlitebrowser) to verify the dog image urls are actually getting saved, and they are.

Here is the current code:

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
            button { id: "skip", onclick: move |_| img_src.restart(), "skip" }
            button {
                id: "save",
                onclick: move |_| async move {
                    let current = img_src.cloned().unwrap();
                    img_src.restart();
                    _ = save_dog(current).await;
            },
            "save!" }
        }
    }
}

#[derive(serde::Deserialize)]
struct DogApi {
    message: String,
}

#[server]
async fn save_dog(image: String) -> Result<(), ServerFnError> {
    DB.with(|f| f.execute("INSERT INTO dogs (url) VALUES (?1)", &[&image]));
    Ok(())
}

// Databse should only be on the server side
#[cfg(feature = "server")]
thread_local! {
    pub static DB: rusqlite::Connection = {
        // open "hotdog.db" database
        let connection = rusqlite::Connection::open("hotdog.db").expect("Failed to open database");

        // Create the "dogs" table if it doesn't already exist
        connection.execute_batch(
            "CREATE TABLE IF NOT EXISTS dogs (
                id INTEGER PRIMARY KEY,
                url TEXT NOT NULL
            );",
        ).unwrap();

        // Return the connection
        connection
    }
}
```

## [Routing and Structure](https://dioxuslabs.com/learn/0.6/guide/routing)

Nice! Now the tutorial is going over better structuring the code. It is going to look like this:

```
├── Cargo.toml
├── assets
│   └── main.css
└── src
    ├── backend.rs
    ├── components
    │   ├── favorites.rs
    │   ├── mod.rs
    │   ├── nav.rs
    │   └── view.rs
    └── main.rs
```

I'm essentially creating all these files. Then moving the server-specific code to `backend.rs`. Then the UI components that are still in `main.rs` will get moved into the proper files in `src/components/`.

The documentation notes that, if this project were started with the `Jumpstart` or `Workplace` template, then a lot of this scaffolding would already exist.

I'm a little confused at this point because the code in my `main.rs` is not working. I can't access the `App` component in `views.rs`. The documentation is moving on to adding a router, so I am probably getting ahead of myself.

I need to add the dioxus `router` feature.

```bash
cargo add dioxus --features router
```

Then I follow the document to make a Route enum and an `app` function that leverages the router. The documentation is getting a lot harder to follow at this point. For example, I add the default route for when a page is not found, but `PageNotFound` is not found in the scope. Does that mean I need to create my own, or is this something I need ot add with a use statement somewhere?

I keep pressing on. Lots of times I get ahead of myself. Perhaps all will be answered soon. The next screenshot does not have `PageNotFound`. It is also leveraging `DogView` from components, unlike the earlier code it showed with

```rust
fn DogView() -> Element {
    todo!()
}
```

I also had to go into `src/backend.rs` and make `save_dog` public. Then I had to go into `src/components/view.rs` and make `DogView` public. I also needed to add `use crate::backend;` and then call the function as `backend::save_dog(current).await`. I am not sure at this point if all of that is the correct thing to do, but the application is working again. I even verify the dog image urls are still being written properly to the database. Huzzah!

Now it is time to add a NavBar. This is pretty straight-forward. I just follow the documentation.

Then I add backend code to get the last 10 saved dogs, and the view code in `favorites.rs` that uses it. It is pretty slick how this all ties together so nicely. There must be a major downside somewhere...

The tutorial is almost over.

Here is what the code looks like now.

`cargo.toml`
```toml
[package]
name = "hot_dog"
version = "0.1.0"
authors = ["Brian Bustin <_@_._>"]
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
dioxus = { version = "0.6.0", features = ["fullstack", "router"] }
reqwest = { version = "0.12.15", features = ["json"] }
serde = { version = "1.0.219", features = ["derive"] }
rusqlite = {version = "0.34.0", optional = true }

[features]
web = ["dioxus/web"]
desktop = ["dioxus/desktop"]
mobile = ["dioxus/mobile"]
server = ["dioxus/server", "dep:rusqlite"]

[profile]

[profile.wasm-dev]
inherits = "dev"
opt-level = 1

[profile.server-dev]
inherits = "dev"

[profile.android-dev]
inherits = "dev"

```

`dioxus.toml`
```toml
[application]

[web.app]

# HTML title tag content
title = "hot_dog"

# include `assets` in web platform
[web.resource]

# Additional CSS style files
style = []

# Additional JavaScript files
script = []

[web.resource.dev]

# Javascript code file
# serve: [dev-server] only
script = []
```

`src/main.rs`
```rust
mod backend;
mod components;

use components::DogView;
use components::Favorites;
use components::NavBar;

use dioxus::prelude::*;

fn main() {
    dioxus::launch(app);
}

#[derive(Routable, Clone, PartialEq)]
enum Route {
    #[layout(NavBar)]
    #[route("/")]
    DogView,

    #[route("/favorites")]
    Favorites,
}

fn app() -> Element {
    rsx! {
        document::Stylesheet { href: asset!("/assets/main.css") }
        Router::<Route> {}
    }
}
```

`src/backend.rs`
```rust
use dioxus::prelude::*;

#[server]
pub async fn save_dog(image: String) -> Result<(), ServerFnError> {
    DB.with(|f| f.execute("INSERT INTO dogs (url) VALUES (?1)", &[&image]));
    Ok(())
}

// Query the database and return the last 10 dogs and their url
#[server]
pub async fn list_dogs() -> Result<Vec<(usize, String)>, ServerFnError> {
    let dogs = DB.with(|f| {
        f.prepare("SELECT id, url FROM dogs ORDER BY id DESC LIMIT 10")
            .unwrap()
            .query_map([], |row| Ok((row.get(0)?, row.get(1)?)))
            .unwrap()
            .map(|r| r.unwrap())
            .collect()
    });

    Ok(dogs)
}

// Database should only be on the server side
#[cfg(feature = "server")]
thread_local! {
    pub static DB: rusqlite::Connection = {
        // open "hotdog.db" database
        let connection = rusqlite::Connection::open("hotdog.db").expect("Failed to open database");

        // Create the "dogs" table if it doesn't already exist
        connection.execute_batch(
            "CREATE TABLE IF NOT EXISTS dogs (
                id INTEGER PRIMARY KEY,
                url TEXT NOT NULL
            );",
        ).unwrap();

        // Return the connection
        connection
    }
}
```

`src/components/favorites.rs`
```rust
use crate::backend;
use dioxus::prelude::*;

#[component]
pub fn Favorites() -> Element {
    // Create a pending resource that resolves to the list of dogs from the backend
    // Wait for the favorites list to resolve with `.suspend()`
    let favorites = use_resource(backend::list_dogs).suspend()?;

    rsx! {
        div { id: "favorites",
            div { id: "favorites-container",
                for (id, url) in favorites().unwrap() {
                    // Render a div for each photo using the dog's ID as the list key
                    div {
                        key: id,
                        class: "favorite-dog",
                        img { src: "{url}" }
                    }
                }
            }
        }
    }
}
```

`src/components/nav.rs`
```rust
use crate::Route;
use dioxus::prelude::*;

#[component]
pub fn NavBar() -> Element {
    rsx! {
        div { id: "title",
            Link { to: Route::DogView,
                h1 { "🌭 HotDog! " }
            }
            Link { to: Route::Favorites, id: "heart", "♥️" }
        }
        Outlet::<Route> {}
    }
}
```

`src/components/view.rs`
```rust
use crate::backend;
use dioxus::prelude::*;

#[component]
fn Title() -> Element {
    rsx! {
        div { id: "title",
            h1 { "HotDog! 🌭" }
        }
    }
}

#[component]
pub fn DogView() -> Element {
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
            button { id: "skip", onclick: move |_| img_src.restart(), "skip" }
            button {
                id: "save",
                onclick: move |_| async move {
                    let current = img_src.cloned().unwrap();
                    img_src.restart();
                    _ = backend::save_dog(current).await;
            },
            "save!" }
        }
    }
}

#[derive(serde::Deserialize)]
struct DogApi {
    message: String,
}
```

`src/components/mod.rs`

```rust
mod favorites;
mod nav;
mod view;

pub use favorites::*;
pub use nav::*;
pub use view::*;
```

## [Bundling](https://dioxuslabs.com/learn/0.6/guide/bundle)

I might do this later, but I am not sure yet. It allows building an iOS or Android application and testing them.

# Conclusion

[Dioxus](learn/0.6/guide/bundle) is a pretty neat framework. I am not enough of a fullstack developer to recommend it or not. It seems like the mixing of server and frontend could get you into trouble. On the other hand, it is awfully convenient.

Rust is great for developing efficient runtimes. If Dioxus can be used to create an application with minimal hosting costs, while making it easier to get everything going, that sounds great and I am all for it.

Long ago, for work, I rewrote one of our small web APIs in Rust. This was for a lunch and learn talk I gave. The original Java version took several seconds to spin up and was guzzling down over 200MB of RAM. My functionally-identical version in Rust spun up in milliseconds and consumed around 4MB of RAM. If the service were used at scale, switching to Rust could have saved a significant amount of money. The talk went over so well, that I gave it again for my colleagues in India. Unfortunately, nothing ever came of it and we stuck to Java.
