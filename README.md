![Deploy to Firebase Hosting on merge](https://github.com/EkeMinusYou/mineo-engine/workflows/Deploy%20to%20Firebase%20Hosting%20on%20merge/badge.svg)

# mineo-engine

## Local development

Install the latest Hugo release (tested with 0.169.0; PaperMod requires 0.158.0 or newer), then initialize the theme and start the server:

```sh
git submodule update --init --recursive
hugo server
```

To run the same build check as CI:

```sh
hugo --panicOnWarning
```

CI runs on Ubuntu with Hugo Extended 0.169.0 from its official GitHub release and deploys with Firebase CLI 15.34.0. Both versions are pinned in `.github/workflows/firebase-hosting-merge.yml`; the Firebase CLI pin avoids the [Hosting Service Usage permission regression](https://github.com/firebase/firebase-tools/issues/11335).
