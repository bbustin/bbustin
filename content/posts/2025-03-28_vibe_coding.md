+++
title = "Vibe Coding"

[taxonomies]
tags = ["AI", "Agents", "Coding", "Vibe Coding"]
+++
# Change to active present tense

I have repeatedly heard the term [Vibe Coding](https://en.wikipedia.org/wiki/Vibe_coding) since last month. Having worked in Software Engineering for over 13 years, I naturally have a bit of skepticism. Determined not to fall into the curmudgeon trap, I am going to try out vibe coding and see what it is all about.

<!-- more -->

# Tools
To make this work, I am going to need to use tools that are made for vibe coding. There are a bunch to choose from, and I am not an expert... yet.

My IDE of choice for personal projects is currently [Zed](https://zed.dev/). It is not really set up for vibe coding yet. There is a way to enable some aspects of it by adding `"enable_experimental_live_diffs": true` to the `assistant` section in `settings.json`, but I want to give this a fair shake.

There are two main Vibe Coding tools that caught my eye:

1. [Cursor](https://www.cursor.com/)
2. [Cline](https://cline.bot/)

Cursor is a complete IDE in and of itself. It looks like it is effectively tied to a subscription after a two week free trial. Cline is an Open Source plugin for VSCode.

I feel like Cursor will be more likely to just work. Since I'm trying to give this a fair shake  I will use Cursor. I am more inclined to like the idea of Cline better due to its open source nature. It also looks like with Cline you pay the AI providers directly rather than going through a middleman. If I like Vibe Coding in Cursor, I will try Cline.

# What to create and the prompt I am going to use

Since I am currently looking for a new job, I think a Job Application tracking tool would be helpful. It would be nice as a GUI application for now. I want it created in Rust. I know Rust is not the ideal programming language for GUI development at this point, but there are several UI libraries. I'm sure this would work better in Python or JavaScript as a website.

## Bare minimum functionality {#bare_minimum}

Here is my prompt for the minimum functionality.

```markdown
Create a GUI application in Rust for tracking Job applications. This application should have at least the following fields, but feel free to add additional relevant fields:

* Date submitted
* Job Description URL
* Title
* Response Date
* Notes

Pressing the tab button on the keyboard should change focus to the next field and allow typing in a response.

The date fields should use a date picker. `Date submitted` should default to today's date.

Submitted job applications should be persisted. The method of persistence is up to you.

Previously submitted job applications should be visible and they should be able to be selected and edited.
```

## Nice to have functionality

If this works, it would be nice to have the application go out to the Job description URL and automatically fill in the title field.

# Cursor

I install [Cursor](https://www.cursor.com/) and create a blank directory for the project. Then I open Cursor to that directory and am ready to begin.

I paste in the [prompt for the bare minimum application](#bare_minimum). This is really cool. It has created a `main.rs` file with the code and a `cargo.toml` file with the dependencies. I just click on the `Apply all` button and those files are created like magic.

Now I ask the AI to `run the application`. It gives me a bunch of steps I don't need to do. I was hoping it would run the application so it could see any errors and automatically correct them. Let me try again `generate a command to run this application` (it should just create `cargo run`). This is also completely unhelpful. It creates some crazy commands that will echo the contents of the various files onto the machine. The last command; however, is `cargo run` and there is a little button that says `Run`. Click.

It just ran it in the terminal. I could have done that. The application will not compile. There is a button in the chat panel called `Add context`. When I click on that, it lets me add context from the terminal. Now I tell the AI that the application will not compile.

It claims to have figured out the problem. I click the `Apply all` and then `Accept all` buttons without really paying attention. That is what Vibe Coding is all about, right?

Running it again in the terminal. Will it compile? Nope. Now I am going to repeat the last step by adding the terminal context in again. Told it to apply the changes it made. Will it work now?

Nope. There are still syntax errors. Let me see if I can get the AI to try to compile and then see the errors automatically. If it can loop through this, that would make iteration faster.

> The application will not compile. Can you please compile the application and then fix any errors you find until the application compiles properly?

It decided to create a fresh project!? Then says it will then iteratively compile and fix things. It did not actually try to compile and fix things. I have seen demos on YouTube where it did this, so I know it is possible...

I updated the context to have the whole codebase. Then I prompted it:

> Please compile and test this code.

It made a bunch of changes again which I applied. Then it ended with the `cargo run` command. So I clicked `Run`. Alas, it ran it in the terminal and still will not compile. I can't say I am terribly impressed.

It looks like it was in `Ask` mode. Let me try `Agent` mode. I think that will allow for the AI to "take charge" a little more. Now it is doing what I expected. It is building and fixing. This is nice.

It did two rounds of compile and fix and now the application came up. It allows for tabbing through the fields. It saves the applications and lets you get back to them to edit. The only thing that is not working is the calendar date picker.

> The application compiled and came up, but the datepicker is not displaying properly for the two date fields. On `Date Submitted` it does not show at all. On `Response Date` it shows what appears to be multiple instances of the widget.

It purports to fix it, but the same exact behavior occurs I run it.

> The datepicker issue is not fixed.

Then it does not compile, so the AI agent immediately goes about trying to fix it. The application then compiles, but now neither date picker will work.

> Now the date picker does not work for either date field.

It fixes it so something pops up, but it is not really the whole date picker.

> It now pops up something when I click on the button, but it is not a whole date picker. I have attached a screenshot.

It then fails to compile, so the AI immediately goes about trying to fix it. Now something pops up with the current year, month, and day as well as a button that says `Today`, but it does not allow actually picking a date.

> The date picker pops up and has the current day, month, and year. It also has a today button. It is missing the days in the month to select from.

I send the prompt above with a screenshot. This is starting to feel more tedious than just trying to do it myself. It now lets me choose any day from this month, but the arrows next to the month to change it just cause the date picker to go away.

> The date picker now allows for choosing any day this month. It does not allow changing the month. The arrows next to the month only cause the date picker to disappear.

It makes some ineffective changes.

> The same problem is occurring with the date picker.

It makes some changes and now it will not compile. It then goes ahead and makes some additional changes. It still does not compile, so it makes additional changes. Now it compiles. The arrow buttons next to the month do not cause the date picker to disappear, but they also do not do anything.

> The arrows near the month no longer cause the date picker to go away when clicked. These buttons are supposed to change the month. The one on the left of the month should change it to the month that comes before the currently displayed month. The button after the month should change it to the month that comes after the currently displayed month.

It makes some changes, but the same problem is occurring.

> The previous problem is still occurring.

Now it goes to change the popup behavior, but that is not the problem. It looks like it is trying to put back the old broken code where the widget kept disappearing. I reject the changes.

> The popup behavior is not the problem. The issue is that the month in the date picker can not be changed.

It still does not work. I am going to stop working on it at this point.

Here is the `cargo.toml` it generated:

```toml
[package]
name = "job_tracker"
version = "0.1.0"
edition = "2021"

[dependencies]
eframe = "0.24.1"
egui = "0.24.1"
egui_extras = { version = "0.24.1", features = ["datepicker"] }
chrono = { version = "0.4", features = ["serde"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

There is a newer Rust edition that should be used on a new project: `2024`.
The latest `eframe`, `egui`, and `egui_extras` version is `0.31.1`. We are starting a project out with a dependency version that is over a year out of date.

Here is the `main.rs` it generated:

```rust
use chrono::{DateTime, Local, Datelike, NaiveDate, Month};
use eframe::egui::{self, Pos2};
use eframe::egui::Widget;
use egui_extras::DatePickerButton;
use serde::{Deserialize, Serialize};
use std::fs;

struct DatePicker {
    year: i32,
    month: u32,
    day: u32,
    changed_month: bool,
    popup_id: egui::Id,
}

impl DatePicker {
    fn from_date(date: NaiveDate, id: impl std::hash::Hash) -> Self {
        Self {
            year: date.year(),
            month: date.month(),
            day: date.day(),
            changed_month: false,
            popup_id: egui::Id::new(id),
        }
    }

    fn show(&mut self, ui: &mut egui::Ui, button_pos: Pos2) -> bool {
        let mut changed = false;

        egui::Window::new("date_picker")
            .fixed_pos(button_pos + egui::vec2(0.0, 25.0))
            .fixed_size([220.0, 240.0])
            .title_bar(false)
            .frame(egui::Frame::popup(ui.style()))
            .show(ui.ctx(), |ui| {
                ui.vertical(|ui| {
                    // Month and Year selector
                    ui.horizontal(|ui| {
                        let prev_button = egui::Button::new("◀");
                        if ui.add(prev_button).clicked() {
                            if self.month == 1 {
                                self.month = 12;
                                self.year -= 1;
                            } else {
                                self.month -= 1;
                            }
                            self.changed_month = true;
                        }

                        ui.label(format!("{} {}",
                            Month::try_from(self.month as u8).unwrap_or(Month::January).name(),
                            self.year
                        ));

                        let next_button = egui::Button::new("▶");
                        if ui.add(next_button).clicked() {
                            if self.month == 12 {
                                self.month = 1;
                                self.year += 1;
                            } else {
                                self.month += 1;
                            }
                            self.changed_month = true;
                        }
                    });

                    // Calendar grid
                    egui::Grid::new("calendar").spacing([4.0, 4.0]).show(ui, |ui| {
                        // Day headers
                        for day in ["Su", "Mo", "Tu", "We", "Th", "Fr", "Sa"].iter() {
                            ui.label(*day);
                        }
                        ui.end_row();

                        // Get the first day of the month and total days
                        let first_day = NaiveDate::from_ymd_opt(self.year, self.month, 1)
                            .unwrap_or_else(|| NaiveDate::from_ymd_opt(self.year, 1, 1).unwrap());
                        let first_weekday = first_day.weekday().num_days_from_sunday();
                        let days_in_month = NaiveDate::from_ymd_opt(self.year, self.month, 1)
                            .map(|d| {
                                let next_month = d.checked_add_months(chrono::Months::new(1))
                                    .unwrap_or(d);
                                d.signed_duration_since(next_month).num_days().abs() as u32
                            })
                            .unwrap_or(31);

                        // Empty cells before first day
                        for _ in 0..first_weekday {
                            ui.label("");
                        }

                        // Days of the month
                        for day in 1..=days_in_month {
                            let is_selected = day == self.day && !self.changed_month;
                            let text = day.to_string();
                            if ui.selectable_label(is_selected, text).clicked() {
                                self.day = day;
                                self.changed_month = false;
                                changed = true;
                            }

                            if (day + first_weekday) % 7 == 0 {
                                ui.end_row();
                            }
                        }
                    });

                    // Today button
                    ui.horizontal(|ui| {
                        if ui.button("Today").clicked() {
                            let today = Local::now().date_naive();
                            self.year = today.year();
                            self.month = today.month();
                            self.day = today.day();
                            self.changed_month = false;
                            changed = true;
                            ui.ctx().request_repaint(); // Force UI update
                        }
                    });
                });
            });

        // Adjust day if it's beyond the days in the current month
        if self.changed_month {
            if let Some(_) = NaiveDate::from_ymd_opt(self.year, self.month, self.day) {
                // Date is valid, keep it
            } else {
                // Date is invalid, adjust to last day of month
                if let Some(last_day) = NaiveDate::from_ymd_opt(self.year, self.month, 1)
                    .and_then(|d| d.checked_add_months(chrono::Months::new(1)))
                    .map(|next_month| next_month.pred_opt().unwrap_or(next_month).day())
                {
                    self.day = last_day;
                }
            }
        }

        changed
    }

    fn date(&self) -> Option<NaiveDate> {
        NaiveDate::from_ymd_opt(self.year, self.month, self.day)
    }
}

#[derive(Serialize, Deserialize, Clone)]
struct JobApplication {
    date_submitted: DateTime<Local>,
    url: String,
    title: String,
    response_date: Option<DateTime<Local>>,
    notes: String,
}

impl Default for JobApplication {
    fn default() -> Self {
        Self {
            date_submitted: Local::now(),
            url: String::new(),
            title: String::new(),
            response_date: None,
            notes: String::new(),
        }
    }
}

#[derive(Default)]
struct JobTrackerApp {
    applications: Vec<JobApplication>,
    current_application: JobApplication,
    selected_index: Option<usize>,
    save_path: String,
}

impl JobTrackerApp {
    fn new(_cc: &eframe::CreationContext<'_>) -> Self {
        let save_path = "job_applications.json".to_string();
        let applications = if let Ok(data) = fs::read_to_string(&save_path) {
            serde_json::from_str(&data).unwrap_or_default()
        } else {
            Vec::new()
        };

        Self {
            applications,
            current_application: JobApplication::default(),
            selected_index: None,
            save_path,
        }
    }

    fn save_applications(&self) {
        if let Ok(json) = serde_json::to_string_pretty(&self.applications) {
            let _ = fs::write(&self.save_path, json);
        }
    }
}

impl eframe::App for JobTrackerApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("Job Application Tracker");

            // Input form
            ui.group(|ui| {
                ui.heading("New Application");

                // Date submitted
                ui.horizontal(|ui| {
                    ui.label("Date Submitted:");
                    let date_text = self.current_application.date_submitted.format("%Y-%m-%d").to_string();
                    let button_response = ui.button(&date_text);
                    let popup_id = ui.make_persistent_id("date_submitted_popup");

                    if button_response.clicked() {
                        ui.memory_mut(|mem| mem.toggle_popup(popup_id));
                    }

                    let is_open = ui.memory(|mem| mem.is_popup_open(popup_id));
                    if is_open {
                        let mut date_picker = DatePicker::from_date(
                            self.current_application.date_submitted.date_naive(),
                            popup_id
                        );
                        if date_picker.show(ui, button_response.rect.left_bottom()) {
                            if let Some(date) = date_picker.date() {
                                self.current_application.date_submitted = DateTime::from_naive_utc_and_offset(
                                    date.and_time(Local::now().time()),
                                    *Local::now().offset(),
                                );
                                ui.memory_mut(|mem| mem.close_popup());
                            }
                        }
                    }
                });

                // URL
                ui.horizontal(|ui| {
                    ui.label("Job URL:");
                    ui.text_edit_singleline(&mut self.current_application.url);
                });

                // Title
                ui.horizontal(|ui| {
                    ui.label("Job Title:");
                    ui.text_edit_singleline(&mut self.current_application.title);
                });

                // Response date
                ui.horizontal(|ui| {
                    ui.label("Response Date:");
                    let response_text = self.current_application.response_date
                        .map_or("Not yet".to_string(), |d| d.format("%Y-%m-%d").to_string());
                    let button_response = ui.button(&response_text);
                    let popup_id = ui.make_persistent_id("response_date_popup");

                    if button_response.clicked() {
                        ui.memory_mut(|mem| mem.toggle_popup(popup_id));
                    }

                    let is_open = ui.memory(|mem| mem.is_popup_open(popup_id));
                    if is_open {
                        let mut date_picker = DatePicker::from_date(
                            self.current_application.response_date
                                .unwrap_or_else(Local::now)
                                .date_naive(),
                            popup_id
                        );
                        if date_picker.show(ui, button_response.rect.left_bottom()) {
                            if let Some(date) = date_picker.date() {
                                self.current_application.response_date = Some(DateTime::from_naive_utc_and_offset(
                                    date.and_time(Local::now().time()),
                                    *Local::now().offset(),
                                ));
                                ui.memory_mut(|mem| mem.close_popup());
                            }
                        }
                    }
                });

                // Notes
                ui.horizontal(|ui| {
                    ui.label("Notes:");
                    ui.text_edit_multiline(&mut self.current_application.notes);
                });

                if ui.button("Submit").clicked() {
                    if let Some(index) = self.selected_index {
                        self.applications[index] = self.current_application.clone();
                    } else {
                        self.applications.push(self.current_application.clone());
                    }
                    self.current_application = JobApplication::default();
                    self.selected_index = None;
                    self.save_applications();
                }
            });

            // List of applications
            ui.group(|ui| {
                ui.heading("Submitted Applications");
                egui::ScrollArea::vertical().show(ui, |ui| {
                    for (index, app) in self.applications.iter().enumerate() {
                        ui.horizontal(|ui| {
                            if ui.selectable_label(
                                self.selected_index == Some(index),
                                format!(
                                    "{} - {}",
                                    app.date_submitted.format("%Y-%m-%d"),
                                    app.title
                                ),
                            ).clicked() {
                                self.selected_index = Some(index);
                                self.current_application = app.clone();
                            }
                        });
                    }
                });
            });
        });
    }
}

fn main() -> eframe::Result<()> {
    let native_options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default()
            .with_inner_size([800.0, 600.0]),
        ..Default::default()
    };

    eframe::run_native(
        "Job Application Tracker",
        native_options,
        Box::new(|cc| Box::new(JobTrackerApp::new(cc))),
    )
}
```

I do not have any experience with the `egui framework`, but this code seems somewhat complex for what it is. Especially around the date picker. Egui already has a [built-in date picker](https://docs.rs/egui-datepicker/latest/egui_datepicker/) with the desired behaviors built in. All you need to do is use the following code (from the documentation):

```rust
fn draw_datepicker(&mut self, ui: &mut Ui) {
    ui.add(DatePicker::new("super_unique_id", &mut self.date));
}
```

# Conclusion

My sample size is incredibly small and I did dictate the programming language used. It is also not a web app, which I suspect most projects Vibe Coding is used for are. Please take this conclusion with a grain of salt.

Vibe Coding, from what I can see, is a way of making something quickly. What you end up with might be convoluted code using libraries that are a year out of date. If you don't look at the code, it might appear you have made something good.

It reduces the friction to get started on something. If what you created is a success, you're going to have to maintain what the AI created for you. Developers spend more time reading code than writing it. It means the time saved upfront might lead to a lot more time down the road. Unless, of course, the AI can maintain the code as well (which is not out of the realm of possibility).

The experience I had here was not enjoyable. I was not "vibing". It felt like it would have been easier to just do it myself. I do not particularly like the code it generated. I still have no idea how to use the egui library, nor do I even know if it is the best gui library for the job.

I love learning. Using an AI assistant while coding helps me learn and can save time. Especially when I turn off the distracting autocomplete suggestions from it and just chat in a sidebar. I use the AI more like a research assistant who has read the manual.

Vibe coding feels more like throwing darts at a dartboard with a blindfold on. I get to take the blindfold off after each throw to see where I ended up, but then I need to take another shot in the dark.

I could have probably gotten a better result with improved prompts or letting the AI make different choices about the language to use. Frankly, though, if we all adapt to what the AI is trained the best to do it is like reversion to the mean. Lots of AI written software will likely all be using the same language, out of date libraries, and have the same security vulnerabilities.

Vibe coding, in its current form, is not for me.
