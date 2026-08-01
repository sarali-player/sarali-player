# Verify a Sarali release

The release folder contains the one-file player, a signed-or-unsigned release manifest, SHA-256 checksums, license/notices, and SPDX and CycloneDX SBOMs. Verification never requires access to music or learner JSON.

## 1. Verify checksums

Run the command from inside the release folder.

Windows PowerShell:

```powershell
Get-Content SHA256SUMS.txt | ForEach-Object {
    $hash, $file = $_ -split '  ', 2
    if ((Get-FileHash -Algorithm SHA256 -LiteralPath $file).Hash.ToLowerInvariant() -ne $hash) {
        throw "Checksum mismatch: $file"
    }
}
```

macOS:

```sh
shasum -a 256 -c SHA256SUMS.txt
```

Linux:

```sh
sha256sum -c SHA256SUMS.txt
```

A matching checksum proves that files agree with this folder's checksum list; by itself it does not prove who published the folder.

## 2. Check the release manifest

`sarali-release.json` names the expected player file, version, byte size, and SHA-256. Sarali's optional shared-folder update check uses only that manifest and the named player file. It does not use the network or replace the running file.

## 3. Verify signed provenance when present

Unsigned bundles have `"provenance": null` in the release manifest. A release created with the optional GitHub attestation contains `sarali-player.attestation.json`.

Online verification with GitHub CLI:

```sh
gh attestation verify sarali-player.html --repo OWNER/REPOSITORY
```

Offline verification using the included Sigstore bundle:

```sh
gh attestation verify sarali-player.html --bundle sarali-player.attestation.json
```

Signed provenance adds publisher/workflow authenticity to the checksum integrity checks. A private repository does not need to be made public to distribute the player, although GitHub's support for private-repository attestations depends on the repository's plan.
