+++
title = "🐴 Creating Horse Hot or Not 🐎"

[taxonomies]
tags = ["Coding", "Rust", "Dioxus"]
+++

I already [followed the Dioxus tutorial and created a GUI application](@/posts/2025-04-01_creating_a_gui_application_in_rust_using_dioxus.md). Now I needed a pet project to learn even more about Dioxus. It couldn't be too complex as I am still learning the framework. It had to have some similarity to the application from the tutorial so I would not get too stuck and could reference it. I did not want to touch user authentication just yet.

My idea is [Horse Hot or Not](https://hhn.bustin.tech). It is a bit of a spoof on an old website that, misogynistically, was used to rate women as being "Hot" or "Not". Making this site is not an endorsement of the original site. The subject here will be horses.

<!-- more -->

Here is what the site should do, at a minimum:

* Rating page
  * Show a picture of a random horse and "Hot" and "Not" buttons
  * Allow user to click one of the buttons to vote on the horse shown
* Leaderboard page
  * Show images of horses starting with the horse that has the highest net number of hot votes (hot - not)
* Keep a person from being able to rate the same horse multiple times
* Not store user-identifiable data (I am a big fan of privacy and do not want to have more information than is needed)

The last two items are the ones I spent the most time thinking about before I started coding. The system to prevent multiple votes on a horse does not need to work perfectly. This is not a high stakes site. How could I prevent the user from voting twice while not storing any identifiable information about them?

Then the idea strikes. I can use a hash. If I store the hash, I can not directly access the identifying information. It would take brute forcing or a [rainbow table](https://en.wikipedia.org/wiki/Rainbow_table). That is probably good enough, but what data should I hash? How about `user_ip:horse_id`. This would allow me to determine if the user voted for a particular horse before because I will have their IP address in the request and will know the id of the horse they are submitting a rating for. I can look up if the hash exists in a database table first, if not, record the vote and then add the hash to a table.

Using the IP address as the user identifier is far from perfect. Here are just some of the problems with this approach:

* If the site is accessed from a phone while on WiFi, then the user leaves the location and is on a cellular data connection, their IP address will change
* IPv4 connections behind a NAT router will all come from the same IP address, even if there are multiple people or devices trying to vote. This essentially locks the household down to only being able to vote once for a horse.
* IP addresses do not correspond to individuals, but more to devices.

I could combine this with a browser cookie, and I am sure there are other approaches I could take, but this is good enough for now.

I get the project all set up and start coding.

# Challenges

## Getting to the Axum configuration

Dioxus uses Axum under the hood, but it does a great job of hiding it. This is a smart approach because there is no need to shoulder the cognitive load of Axum in most cases. Axum does not include much information about the request within it unless configured to do so. I need the IP address the request is being made from.

I also need to be able to serve static images, but I can not seem to find a way to do that directly from within Dioxus. There is a way to include assets, but that bundles them. I want this to be all self-contained without the need for a CDN or different server setup.

[The tutorial shows how to get to the Axum configuration](https://dioxuslabs.com/learn/0.6/guide/backend/#server-functions-an-inline-rpc-system). The problem I keep running up against is on this line:

```rust
axum::Router::new().serve_dioxus_application(config, app);
```

`serve_dioxus_application` simply does not exist. I eventually find [issue 3479](https://github.com/DioxusLabs/dioxus/issues/3479). When I added axum to my `cargo.toml`, I had used a version that was newer than the one supported by Dioxus. Once I change it to version `0.7.9`, it starts working. This took me a while to figure it out, but will no [longer be a problem soon](https://github.com/DioxusLabs/dioxus/pull/3820).

To get the connection information in the request, I add `let router = router.into_make_service_with_connect_info::<SocketAddr>();`.

To serve the static images, I use the `tower_http` `ServeDir` service and add it to the Axum router. `.nest_service("/static/horses", ServeDir::new("public/static/horses"));`.

The `main` function in `main.rs` now looks like this:

```rust
// The entry point for the server
#[cfg(feature = "server")]
#[tokio::main]
async fn main() {
    // dependencies that only apply to the server
    use std::net::SocketAddr;
    use tower_http::services::ServeDir;

    // Connect to dioxus' logging infrastructure
    dioxus::logger::initialize_default();

    // Get the address the server should run on. If the CLI is running, the CLI proxies fullstack into the main address
    // and we use the generated address the CLI gives us
    let address = dioxus::cli_config::fullstack_address_or_localhost();

    // Set up the axum router
    let router = axum::Router::new()
        // You can add a dioxus application to the router with the `serve_dioxus_application` method
        // This will add a fallback route to the router that will serve your component and server functions
        .serve_dioxus_application(ServeConfigBuilder::default(), App)
        .nest_service("/static/horses", ServeDir::new("public/static/horses"));

    // Finally, we can launch the server
    let router = router.into_make_service_with_connect_info::<SocketAddr>();
    let listener = tokio::net::TcpListener::bind(address).await.unwrap();
    match backend::load() {
        Ok(_) => (),
        Err(e) => panic!("Unable to load: {}", e),
    }
    axum::serve(listener, router).await.unwrap();
}
```

## Limitations tracking rating submissions

The initial idea of using the hashes to represent an individual rating submission for a certain horse is sound. The problem is that it does not allow for getting at some information that could be useful. I decide to hash the IP address, but then have a column for the horse_id, and when the rating was submitted. This does make revealing the user's IP address easier (at least in the case of IPv4) because you could simply create a rainbow table with hashes of all possible IP addresses, and then cross-reference each hash in the database against the rainbow table to reveal the address. That is definitely a downside.

The upside is that it allows for more streamlined database queries. The main issue that convinced me to change course was picking a random horse for a user to rate. It does not make sense to serve the user a horse they have already rated. In that scenario, the user would click a rating button, but the rating would not be successfully submitted.

Sure, I could:

* Get a random horse
* Hash the user's IP with the horse_id
* Look up the hash to see if the user rated this horse
* If they did rate this horse, repeat the previous steps until a horse they have not rated is found

This approach has some obvious disadvantages. The most important one to me is it gets more and more compute-intensive the more horses a user has rated. Imagine there are 6,000 horses and the user has rated 5,999 of them. It is going to need to keep repeating the steps until it finds the 1 horse they have not rated. It is not just compute-intensive, it will also get really sluggish.

With the horse_id decoupled from the hash, I can simply use a query that returns a random horse where the user's hash and the horse_id in question does not exist in the table where per-user rating submissions are recorded. This will not be compute-intensive and performance will be great.

## IPv6

I noticed that I could vote for the same horse if I used my computer and then my phone while both were connected to my home's network. My home network uses IPv4 and IPv6. With IPv6 there is no need to be behind NAT. This means each device has a different public IPv6 address. What I needed to do was to actually hash the /64 network address of the IPv6 address. This, in effect, makes it behave closely to the IPv4 behind NAT scenario. It is not perfect. ISPs could assign a /63 or larger network, but it should cover most home networks.

## UI and css

I am not a UI expert. Getting the UI to be good enough took some work. CSS is great, but there are lots of complexities. This is an area I could use some improvement in.

There are still rough edges on the site. I noticed, for example, on a small mobile screen some text that should be centered is not. I have not tried to fix that yet.

# Conclusion

My second go-round with Dioxus went well. I do like the framework. When researching some of the problems I was having, I saw an active community around the project.

The UI state reactivity reminded me of some of the UI work I did in [Vue.js](https://vuejs.org/) for work. Many of the ideas felt, at least superficially, very similar even if the code looks nothing alike.

Don't forget to check out [Horse Hot or Not](https://hhn.bustin.tech)! 🐎
