# HACS integration

[HACS (Home Assistant Community Store)](https://hacs.xyz/) is a powerful integration that you can utilize to install custom, unsupported integrations, themes and UI plugins.

## Installation

More information regarding installation of HACS can be found on the [Installation page](https://hacs.xyz/docs/installation/prerequisites) of the HACS documentation.

!!! note 
    If you want to use Git pull with HACS, please set up your Git repository (like below) first before installing HACS. You will make your life significantly easier for yourself!

## HACS and Git pull add-on

HACS can get a bit tricky when you are also using the Git pull add-on, like I do. I was facing merge issues and accidental deletions when pulling changes from Git. Over time I finetuned my set up so I can have the best of both worlds.

### `.gitignore`

First of all we need to add a couple of lines of code to our `.gitignore` file located in the root of our Git repository:

```gitignore
# HACS .gitignore
!**/.managed-by-hacs
!.managed-by-hacs
```

Since empty directories are never included in a Git commit, we are going to create these additional rules. Both rules explictly include `.managed-by-hacs` files within our Git repository. Ensuring that we are not going to delete them when committing to Git.

### Directories

Since HACS uses several directories in our Git configuration for it's data, we are going to add several directories for HACS. I listed a partial view of my file tree below.

```tree
.
├── appdaemon
│   └── apps
│       └── .managed-by-hacs
├── configuration.yaml
├── custom_components
│   ├── hacs
│   │   └── .managed-by-hacs
│   └── .managed-by-hacs
├── .gitignore
├── netdaemon
│   └── apps
│       └── .managed-by-hacs
├── python_scripts
│   └── .managed-by-hacs
├── README.md
├── themes
│   └── .managed-by-hacs
└── www
    └── community
        └── .managed-by-hacs
```

As you can see I added several directories and added `.managed-by-hacs` placeholder files in them. This ensures that these otherwise empty direcotires are committed into Git. 

After adding these directories and placeholder files, you can commit these changes to Git. When you have verified your changes in your Home Assistant install, you can now continue installing HACS.

!!! note 
    [`appdaemon`](https://hacs.xyz/docs/categories/appdaemon_apps), [`netdaemon`](https://hacs.xyz/docs/categories/netdaemon_apps) and [`python_scripts`](https://hacs.xyz/docs/categories/python_scripts) are optional and not enabled by default. They are not required if you don't use them!!

### `configuration.yaml`

Although enabling HACS is done through an UI integration, you will need to set up `configuration.yaml` for themes:

```yaml
frontend:
  themes: !include_dir_merge_named themes
```

You will, possibly, also need to set up your `configuration.yaml` for your `custom_components`, but that's up to the integrations that you install.

!!! note 
    Refer to the [`appdaemon`](https://hacs.xyz/docs/categories/appdaemon_apps), [`netdaemon`](https://hacs.xyz/docs/categories/netdaemon_apps) and [`python_scripts`](https://hacs.xyz/docs/categories/python_scripts) documentation on how to setup those.
