+++
title = "Diving into AI"

[taxonomies]
tags = ["Research", "AI", "Nix"]
+++

Artificial Intelligence (AI) is taking the world by storm. It is easy to naysay and dismiss it 
as just a fad. That would be a mistake. I think AI is here to stay and want to learn how it works 
and how to leverage it.

I am very concerned with privacy and not giving companies information for free they can
monetize. This means I need to run all of the models locally. Here I will document what I
have done, and why. I hope this information helps you and also helps future me. 
<!-- more -->

# Computer
To run the models quickly, I need a pretty capable machine to run them on.
I have chosen one with an Apple M4 processor and ample RAM. The main reasons I went
with an M4 are:

* I already use MacOS as my daily driver. I'd rather get a machine I can use for more than
just running AI models.
* The unified memory architecture allows the system to share memory between the processor
and the GPUs. This should allow me to do more with less.

# Ollama
I start my journey with Ollama. It is the local AI gateway drug.

Installation is a breeze:
1. Go to [ollama.com/download](https://ollama.com/download)
2. Download installer
3. Run installer

Running models is simple
* `ollama run model_name`

I try a bunch of models including:
* llama3.2
* llama3.3
* qwen2.5

I find Ollama amazing. The people who worked on this did a fantastic job. It just worked. Perfectly.
The downside for my particular goal of learning more about AI is I do not get to see the
gnarly stuff that makes this all work.

***Note:** Ollama keeps a local history of what you send in `~/.ollama/history`. If you would like to
temporarily disable this, send `/set nohistory` first before anything you do not want saved.
No history will be saved after that for the current session.*

# Getting Into the Weeds
To really see how to leverage local AI, I need to work at a lower level. I want to go through the
[Dive into Deep Learning](https://d2l.ai/) book. The 
[installation instructions](https://d2l.ai/chapter_installation/index.html) involve making global
changes to my machine. I read that different AI/ML libraries often require different versions
of python. I want to be able to create little mini-environments for each of my AI experiments.

I could go the route of [Docker](https://www.docker.com/)/[podman](https://podman.io/) containers,
but I've already done that before. There are also articles detailing the difficulties getting
containers to be able to access GPUs.

## Down the Nix Rabbithole
With my old computer, I leveraged [Homebrew](https://brew.sh/) to install all of the tools
I was going to use. I really liked Homebrew, but it installs things globally. What I need
is a tool which can install things either globally or locally. I am choosing use the
[Nix Package Manager](https://en.wikipedia.org/wiki/Nix_(package_manager)) because it
pretty much does that.

The `nix develop` or `nix-shell` commands can create a bash shell that has certain libraries
available to it. When you exit the shell, poof they're gone. (Well not really, they still
are there taking up disk space somewhere, but your global configuration has not changed.)

### Confusion
I definitely find Nix a bit confusing. At this point in my journey I am starting to wonder if I
bit off more than I can chew. First off, Nix can refer to one of 3 things:

1. [Nix](https://nix.dev/manual/nix/latest/language/index.html), the programming language
2. [Nix](https://en.wikipedia.org/wiki/Nix_(package_manager)), the package manager
3. [NixOS](https://en.wikipedia.org/wiki/NixOS), the operating system

I will use Nix, the package manager which you can get from [nixos.org](https://nixos.org/). Once more, it
is confusing that you get the package manager from a site with a name ending in os. I will put that
aside. In the rest of this post Nix refers to the package manager.

Second, there are some splits in the Nix community. The first is whether to use 
[Nix flakes](https://nixos.wiki/wiki/flakes). The second is whether to use the
[Nix command](https://nixos.wiki/wiki/Nix_command) instead of having different commands 
for each function (ie `nix build` instead of `nix-build`).

Third, there are multiple ways to install it on MacOS:

1. [nixos site](https://nixos.org/download/)
2. [Determinate Nix Installer](https://github.com/DeterminateSystems/nix-installer)

***note:** I do not recommend piping the output from curl into sh. Instead, download the 
       installation script, inspect it, and then run the downloaded copy.*

### Nix Package Manager Installation     
I will use the [Determinate Nix Installer](https://github.com/DeterminateSystems/nix-installer).
The main reason I am choosing this is because it has an uninstaller and looks like it is more
likely to survive OS upgrades. It also enables [Nix flakes](https://nixos.wiki/wiki/flakes) and
[Nix command](https://nixos.wiki/wiki/Nix_command). I plan to leverage flakes. This
might not be the right choice, I am but a Nix n00b, but their promise of "improving reproducibility
of Nix installations" sounds nice. When I first started using [ZFS](https://en.wikipedia.org/wiki/ZFS),
I loved how everything used the same command. It made it easy to discover all the different subcommands 
and their options. This makes Nix command sound appealing.

I am not nuts about how many changes it will make to my system,
but at least they are clearly spelled out and reversible. 

I follow the steps to install Nix using the shell installer without installing determinate.
I did download the script, inspect it, and then run it. This all goes off without a hitch.

### nix-darwin Installation
I decide also to install [nix-darwin](https://github.com/LnL7/nix-darwin/). This lets me
control some of the settings on my computer declaratively using a Nix flake. Doing this
is not really related to setting up a machine for running AI models, but it sounds nice
to be able to make certain system-level changes.

I follow the steps at 
[nixcademy.com/posts/nix-on-macos/](https://nixcademy.com/posts/nix-on-macos/).
and end up with this flake:

```nix,linenos
{
  description = "nix-darwin system flake";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
    nix-darwin.url = "github:LnL7/nix-darwin";
    nix-darwin.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = inputs@{ self, nix-darwin, nixpkgs }:
  let
    configuration = { pkgs, ... }: {
      # List packages installed in system profile. To search by name, run:
      # $ nix-env -qaP | grep wget
      environment.systemPackages =
        [ 
          # Friendly Interactive Shell
          pkgs.fish
          
          # Nice CLI text editor - hx
          pkgs.helix
          
          # Prompt that provides lots of relevant information such as git branch
          pkgs.starship
          
          # Like cat, but built in Rust and has syntax highlighting
          pkgs.bat
          
          # recursive grep written in Rust - rg
          pkgs.ripgrep
        ];

      # Necessary for using flakes on this system.
      nix.settings.experimental-features = "nix-command flakes";

      # Enable alternative shell support in nix-darwin.
      programs.fish.enable = true;

      # Set Git commit hash for darwin-version.
      system.configurationRevision = self.rev or self.dirtyRev or null;

      # Used for backwards compatibility, please read the changelog before changing.
      # $ darwin-rebuild changelog
      system.stateVersion = 5;

      # The platform the configuration will be used on.
      nixpkgs.hostPlatform = "aarch64-darwin";
      
      # allow using touch id for sudo
      security.pam.enableSudoTouchIdAuth = true;
      
      system.defaults = {
        # default to finder column view
        finder.FXPreferredViewStyle = "clmv";
        
        # custom message on login window
        loginwindow.LoginwindowText = "Brian's Laptop";
      };
    };
  in
  {
    # Build darwin flake using:
    # $ darwin-rebuild build --flake .#simple
    darwinConfigurations."<your computer's hostname>" = nix-darwin.lib.darwinSystem {
      modules = [ configuration ];
    };
  };
}
```

I really like this configuration. The only setting that is not working
is trying to default the Finder to column view. My research indicates
Apple took the ability to default to column view out of Sequoia.

If you'd like to use my configuration, you will need to change line 64
to reflect the hostname of your system.

## Dive Into Deep Learning book
The prerequisites are now in place. I download the pytorch code.

```
mkdir d2l-en && cd d2l-en
curl https://d2l.ai/d2l-en-1.0.3.zip -o d2l-en.zip
unzip d2l-en.zip && rm d2l-en.zip
cd pytorch
```

Now I am going to leverage Nix to create a local shell with everything that's
needed. I am using a Nix flake. This may not have been the right choice as
most examples on the web are for nix-shell. But I'm stubborn and I finally get
it working after lots of frustration.

I create a `flake.nix` file in the `pytorch` directory.

```nix
{
	description = "Shell for running d2l Jupyter lab";
	
	inputs = 
		{
			nixpkgs.url = github:nixos/nixpkgs/nixos-unstable;
		};
	
	outputs = { self, nixpkgs, ... }:
		let
			system = "aarch64-darwin";
			pkgs = nixpkgs.legacyPackages.${system};
			pythonVersion = "python39";
		in
		{
			devShells.${system}.default = pkgs.mkShell 
				{
					buildInputs = [ 
						pkgs.${pythonVersion}
						pkgs."${pythonVersion}Packages".pip
						# Install Jupyter using pip, otherwise it cannot access other libraries we install using pip
						# pkgs.jupyter
					];

					shellHook = ''
						python -m venv .venv
						source .venv/bin/activate
						pip install torch==2.0.0 torchvision==0.15.1 d2l==1.0.3
						echo "Running 'jupyter lab'"
						jupyter lab
						echo "exiting nix development shell"
						exit
					'';
					
				};
		};
}
```

It turns out that I probably would have needed much less boiler-plate if I had gone
the `nix-shell` route.

To use this, open Terminal and change to the `pytorch` directory. Then run
`nix develop`. This command, along with the flake takes care of everything. Your default 
web browser will pop up in Jupyter lab and you're ready to follow along.

# Where Next?
Once I have a better sense of the fundamentals I will likely explore various libraries.
Next on my list is [Hugging Face's Transformer library](https://huggingface.co/docs/transformers/main/en/index).
