yo this is my sample haveno build thing
if you're making your own haveno instance please edit the relevant files like so:
TODO: command block of using sed to replace the app id with YOUR_ID_HERE and the http url with theirs

- move config file
- complete all todos
  - [x] make a gpg key with `.gpg` as the home dir and store the b64 in `.flatpakref` (see [gpgkey.sh](./gpgkey.sh) for an example)
  - [x] enter the key id in config.json lines 8 and 36
  - [x] generate a secret and enter it on line 38, and generate a token with that secret
    - the included gentoken project can be used for this:

    ```bash
    cd gentoken
    # assuming the secret is stored in PROJ_DIR/secret.txt
    cargo run -- --base64 --secret-file ../secret.txt --name all > ../token.txt
    cd -
    ```

- `docker compose up -d`
  - you can run `docker compose up` to run it in your terminal and see logs and C-c it when you want, or you can `docker compose logs -f` if you just want to see the logs
- once you see "Started http server", init your stable repo (one time thing):

```bash
# Enter the 'flat-manager' container with an interactive bash shell
docker compose exec -it flat-manager bash

# Update package lists and install 'ostree' inside the container
apt update && apt install -y ostree

# Initialize the 'ostree' repository at the specified path in archive-z2 mode
ostree --repo=/var/run/flat-manager/stable-repo init --mode=archive-z2

# Create a directory '/build-repo' if needed
mkdir -p /build-repo

# Exit
exit
```

- build an app into a temp repo (see <https://docs.flatpak.org/en/latest/flatpak-builder.html>)
- set TOKEN_FILE to location of file containing your **token, not secret**
  - which should be PROJ_DIR/token.txt
- use `./publish.sh <DIR>` to publish that repo
  - keep an eye on the logs to make sure everything goes smoothly
  - if there aren't any errors, press y to "publish" the build
- youre done!

to test on your host:

- flatpak install --from .flatpakref
- flatpak run exchange.haveno.Haveno

notes:

- you wanna share the `.flatpakref` if you want to point people to your app, think of it like a manifest file that says where the app and such data is stored
- tell your users to run `flatpak update exchange.haveno.Haveno` if they wanna update
- the build-dir theoretically could get large over the course of a few builds, i'm not sure if ostree manages the size, doing a down up on the container should remove it
- all of the saved config aside from `.gpg` and `config.json` are stored in docker volumes so see the docs if you wanna edit them
