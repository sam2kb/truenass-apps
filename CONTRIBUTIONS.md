# Contributions

Thank you for your interest in contributing to the Apps Catalog.

If you start working on something, please either open an issue and state that you are working on it,
make a comment on existing issues, or open a PR (even if it's empty!). This will help avoid
duplicate work!

## Library

The library is written in Python and is located in the `/library/2.x.x` directory.
You will rarely need to interact with the library directly.

## New Apps

If you want to add a new app to the catalog, the easier way is
to copy an existing app that is similar to the one you want to add,
then modify it to your needs. You can find usage examples in either,
other apps that use the library, or in the `/library/2.x.x/tests` directory.

For icons and screenshots that we store in our CDN, you can leave the links
in the PR descriptions, and the PR reviewer will upload them and give you the links.

⚠️ Make sure that you are only adding/modifying files under the `/ix-dev` or `/library` directories!
All other files should be ignored, as they are mostly auto-generated.

## Local Testing

You will need to have `docker`, `docker compose` and `python` installed.
Additionally, you need the following Python packages installed:
`pyyaml, psutil, pytest, pytest-cov, bcrypt, pydantic`

If you have `nix-shell` installed, you can run the following command to
start a shell with all the required python packages installed:

```bash
nix-shell -p 'python3.withPackages (ps: with ps; [ pyyaml psutil pytest pytest-cov bcrypt pydantic ])'
```

The directory `test_values` contains test files for each app.
To run a test against a test file, run the following command:

```bash
./.github/scripts/ci.py --app qbittorrent --train community --test-file basic-values.yaml --wait=true
```

`--wait=true` will start the container, and wait until you stop it. It will also show the URL of the web UI (if available).
`--render-only=true` will just print the rendered compose file, without starting the container. (You can pipe the output to a file if you want to save it.)

Both flags are optional. If you don't specify them, it will run the app against the test file.
And stop it as soon as it becomes healthy. It will timeout after 10 minutes if it doesn't become healthy.
If you manually `ctrl+c`, you will have to cleanup the leftover containers.

The above command will also do the following:

- Generate the `item.yaml` file
- Update the contents of the `templates/library/` directory based on the `lib_version` on the `app.yaml` file
- Update the `lib_version_hash` on the `app.yaml` file

ie. if you just want to generate the above, just run the following command, test file can be any, but must exists:

```bash
./.github/scripts/ci.py --app qbittorrent --train community --test-file basic-values.yaml --render-only=true
```

## Testing on TrueNAS System

There is no easy way currently to test on TrueNAS.
As long as it works on your local machine, it should work on TrueNAS.
There are some exceptions for things like devices etc, that might be different.
In such case, let the reviewer know.

`questions.yaml` is run through some validation during CI, but it still needs some manual check from the reviewer.
If you want to test how the rendered compose file will look based on different values entered in the questions.yaml,
you can enter the values you want in one of the test files and render it.

## App structure

```sh
.
├── app.yaml # Contains the app metadata
├── item.yaml # It is automatically generated by the ci.py script
├── ix_values.yaml # Contains static values that are used in the app always
├── questions.yaml # Contains the questions that the user will be asked when deploying the app
├── README.md # Contains a short description of the app
└── templates
    ├── docker-compose.yaml # The template file.
    ├── library
    │   └── base_v2_0_21 # Automatically copied when run the ci.py script based on the lib_version on the app.yaml file
    ├── rendered # This is in the .gitignore and is only used as an intermediate step to deploy the app
    │   └── docker-compose.yaml
    └── test_values # Those are used in CI only, values in there are not used when deploying the app
        ├── basic-values.yaml
        └── hostnet-values.yaml
```

### app.yaml

Things to note:

- `name:` should match the directory name
- `train:` should match the directory above the app directory
- `app_version:` is the version of the container, usually the tag of the `image` if there are multiple kind of images
- `lib_version:` must be one of the available versions in the top level `library` directory. (Do not use v1)

⚠️ Some notes for the test files:

Most apps will use a directory like `/opt/tests/**` for storage in test files.
This is mostly because MacOS whitelists this directory by default. And linux does not have this restriction.

Make sure before running the test, that is not going to mount any directories that you don't want to.

## Notifying Upstream Developers

After your app has been successfully published in the catalog and tested, it is a good idea 
to politely contact the upstream app developers to let them know that TrueNAS now supports 
simple deployment of their app. 
You can provide a quick "how-to" for deploying their app on TrueNAS, to make it easier for their 
team to share with their users. If the developer has a section on their website listing supported 
platforms, suggest they add TrueNAS as one of the platforms capable of running their app 
with easy deployment.
