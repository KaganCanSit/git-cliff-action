# [git-cliff](https://github.com/orhun/git-cliff) action ⛰️

This action generates a changelog based on your Git history using [git-cliff](https://github.com/orhun/git-cliff) on the fly!

## Usage

### Input variables

- `config`: Path of the configuration file. (Default: `"cliff.toml"`)
- `args`: [Arguments](https://github.com/orhun/git-cliff#usage) to pass to git-cliff. (Default: `"-v"`)

### Output variables

- `changelog`: Output file that contains the generated changelog.
- `content`: Content of the changelog.

### Environment variables

- `OUTPUT`: Output file. (Default: `"git-cliff/CHANGELOG.md"`)

### Examples

#### Simple

The following example fetches the whole Git history (`fetch-depth: 0`), generates a changelog in `./CHANGELOG.md`, and prints it out.

```yml
jobs:
  changelog:
    name: Generate changelog
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Generate a changelog
        uses: orhun/git-cliff-action@v1
        id: git-cliff
        with:
          config: cliff.toml
          args: --verbose
        env:
          OUTPUT: CHANGELOG.md

      - name: Print the changelog
        run: cat "${{ steps.git-cliff.outputs.changelog }}"
```

#### Advanced

The following example generates a changelog for the latest pushed tag and sets it as the body of the release.

It uses [svenstaro/upload-release-action](https://github.com/svenstaro/upload-release-action) for uploading the release assets.

```yml
jobs:
  changelog:
    name: Generate changelog
    runs-on: ubuntu-latest
    outputs:
      release_body: ${{ steps.release.outputs.RELEASE_BODY }}
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Generate a changelog
        uses: orhun/git-cliff-action@v1
        id: git-cliff
        with:
          config: cliff.toml
          args: -vv --latest --strip header
        env:
          OUTPUT: CHANGES.md

      - name: Set the release body
        id: release
        shell: bash
        run: |
          r=$(cat ${{ steps.git-cliff.outputs.changelog }})
          r="${r//'%'/'%25'}"     # Multiline escape sequences for %
          r="${r//$'\n'/'%0A'}"   # Multiline escape sequences for '\n'
          r="${r//$'\r'/'%0D'}"   # Multiline escape sequences for '\r'
          echo 'RELEASE_BODY<<EOF' >> $GITHUB_ENV
          echo "$r" >> $GITHUB_ENV
          echo 'EOF' >> $GITHUB_ENV

      # use release body in the same job
      - name: Upload the binary releases
        uses: svenstaro/upload-release-action@v2
        with:
          file: binary_release.zip
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          tag: ${{ github.ref }}
          body: ${{ steps.release.outputs.RELEASE_BODY }}

  # use release body in another job
  upload:
    name: Upload the release
    needs: changelog
    runs-on: ubuntu-latest
    steps:
      - name: Upload the binary releases
        uses: svenstaro/upload-release-action@v2
        with:
          file: binary_release.zip
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          tag: ${{ github.ref }}
          body: ${{ needs.changelog.outputs.release_body }}
```

## Credits

This action is based on [lycheeverse/lychee-action](https://github.com/lycheeverse/lychee-action) and uses [git-cliff](https://github.com/orhun/git-cliff).

## License

GNU General Public License ([v3.0](https://www.gnu.org/licenses/gpl.txt))

## Copyright

Copyright © 2021, [Orhun Parmaksız](mailto:orhunparmaksiz@gmail.com)
