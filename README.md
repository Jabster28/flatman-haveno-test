# Haveno Sample Build

This README provides instructions for setting up a `flat-manager` server for your Haveno instance Flatpaks using Docker.

## Setup Instructions

### Edit Configuration Files

> If you're setting up a new Haveno instance, you may want to edit some strings from this example. You can automate this with `sed`:
>
> ```bash
> sed -i 's/exchange.haveno.Haveno/YOUR_ID_HERE/g' path/to/your/file
> sed -i 's|http://localhost:8080|your-url-here|g' path/to/your/file
> ```
>
> This will also change the string in the README. **Ensure that you have changed the ID and the `bind`ed folder in your Haveno builds too, otherwise users won't be able to install your application.**

1. Rename `sample.config.json` to `config.json`

2. Complete the todos in the project:
   - [x] **Generate a GPG Key:**
     Create a GPG key with `./.gpg` as the gpg home directory and store the base64 encoded key in `.flatpakref`. Refer to the [gpgkey.sh](./gpgkey.sh) script for an example.
   - [x] **Update `config.json`:**
     Enter the new GPG key ID in `config.json` on lines 8 and 36.
   - [x] **Generate and Enter Secret:**
     Generate a secret and enter it on line 38 of `config.json`. Use the `gentoken` project (taken from the `flat-manager` project) to generate a token:

     ```bash
     cd gentoken

     # Assuming the secret is stored in PROJ_DIR/secret.txt
     cargo run -- --base64 --secret-file ../secret.txt --name all > ../token.txt

     cd -
     ```

### Start Services

1. **Bring Up Docker Containers:**

   ```bash
   docker compose up -d
   ```

   You can use `docker compose up` to run the containers in the terminal and view logs. Alternatively, use `docker compose logs -f` to follow the logs without starting the containers in the foreground.

2. **Initialize Stable Repository:**

   Once you see "Started http server", initialize your stable repository (one-time setup):

   ```bash
   # Enter the 'flat-manager' container with an interactive bash shell
   docker compose exec -it flat-manager bash

   # Update package lists and install 'ostree' inside the container
   apt update && apt install -y ostree

   # Initialize the 'ostree' repository at the specified path in archive-z2 mode
   ostree --repo=/var/run/flat-manager/stable-repo init --mode=archive-z2

   # Create a directory '/build-repo' if needed
   mkdir -p /build-repo

   # Exit the container
   exit
   ```

### Build and Publish

1. **Build an App:**
   Build your application into a temporary repository. For details, refer to the [Flatpak Builder documentation](https://docs.flatpak.org/en/latest/flatpak-builder.html).

2. **Set TOKEN_FILE:**
   Set `TOKEN_FILE` to the location of the file containing your token (not the secret), which should be `PROJ_DIR/token.txt`.

3. **Publish the Repository:**

   ```bash
   ./publish.sh <DIR>
   ```

   - Set `FLATMAN_URL` if you're not using the default localhost URL.
   > The `sed` commands mentioned earlier does this for you.
   - Monitor the logs to ensure everything proceeds smoothly.
   - If no errors occur, press `y` to confirm and publish the build. Reminder that anything that isn't published doesn't appear in users' repositories, and will be lost if the container is removed.
   - You're done!

### Testing

To test the installation on your host machine:

```bash
flatpak install --from .flatpakref
flatpak run exchange.haveno.Haveno
```

## Notes

- **`.flatpakref` File:**
  Share the `.flatpakref` file to direct users to your app. It acts as a manifest indicating where the app and related data are stored.

- **Updates:**
  Instruct users to run `flatpak update exchange.haveno.Haveno` to update the app.

- **Repository Size:**
  The build directory may grow large over time. `ostree` manages size to some extent, but running `docker compose down` and `docker compose up` should clear the cache.

- **Configuration Storage:**
  All saved configuration, except for `.gpg` and `config.json`, is stored in Docker volumes. Refer to the Docker documentation if you need to edit these volumes.
