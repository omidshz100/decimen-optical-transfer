# Decimen Optical Transfer

> Transfer files and text between devices using animated QR codes — no network, cable, account, or pairing required.

Decimen Optical Transfer turns a device screen into a one-way optical data channel. The sender displays an endless stream of fountain-coded QR frames, while the receiver scans those frames with its camera and reconstructs the original content.

This edition is customized and maintained by **Omid Shojaeian Zanjani** ([@omidshz100](https://github.com/omidshz100)), with a refreshed light interface, orange visual identity, and simplified branding.

<p align="center">
  <img src="docs/receiving.jpg" width="420" alt="A phone receiving a file from an animated QR code" />
</p>

## Highlights

- Transfers files up to 64 MB through screen-to-camera communication
- Sends plain-text snippets without creating a file first
- Requires no network connection between the two devices
- Uses fountain coding to tolerate missed or dropped QR frames
- Compresses suitable content with gzip before transmission
- Preserves the original filename and media type
- Verifies received files with SHA-256 before download
- Works as an installable PWA after the first visit
- Produces standalone sender and receiver HTML files
- Runs entirely in the browser with no backend or cloud storage

## How to use it

1. Open the **Send** page on the device containing the file.
2. Select a file or switch to text mode and enter a snippet.
3. Open the **Receive** page on a second device.
4. Allow camera access and point the camera at the animated QR code.
5. Keep the devices steady until verification finishes, then save or copy the received content.

For better results, increase the sender's screen brightness and keep the receiving camera stable.

## Local development

### Requirements

- Node.js 18 or newer
- npm
- A modern browser with camera support for the receiving device

### Installation

```bash
git clone https://github.com/omidshz100/decimen-optical-transfer.git
cd decimen-optical-transfer
npm install
npm run dev
```

Vite prints the local and network addresses in the terminal. Open `/send/` on the sending device and `/receive/` on the receiving device.

The development server uses HTTPS because browsers only expose camera access in a secure context. A local certificate warning is therefore expected on the first visit.

## Available commands

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start the HTTPS development server with hot reload |
| `npm run demo` | Start demo mode with only the bundled sample files |
| `npm run build` | Create the production PWA in `dist/` |
| `npm run serve` | Build and preview the production version |
| `npm run build:standalone` | Create standalone sender and receiver HTML files |
| `npm run build:all` | Build both the PWA and standalone versions |
| `npm test` | Run the protocol and unit test suite |

## Standalone mode

Run:

```bash
npm run build:standalone
```

The output is written to `dist-standalone/`:

- `decimen-sender.html` works directly from a local file.
- `decimen-receiver.html` includes the QR decoder and WebAssembly module.

Desktop browsers can usually request camera access from the standalone receiver. Mobile browsers may block camera access for pages opened with `file://`; serving the file over HTTPS or using the installed PWA avoids this limitation.

## How it works

A screen-to-camera connection has no return channel, so the receiver cannot request a missing frame. Decimen solves this with **fountain codes**:

1. The sender divides the payload into blocks.
2. Every QR frame contains the XOR of a deterministic subset of those blocks.
3. The receiver collects any sufficient set of distinct frames, regardless of order.
4. The original payload is reconstructed, decompressed when necessary, and verified.

Each frame also carries stream metadata such as the session identifier, sequence number, block dimensions, payload length, and integrity information. If the sender restarts or its settings change, the receiver automatically recognizes the new session.

QR decoding uses [zxing-wasm](https://github.com/Sec-ant/zxing-wasm) in Web Workers. QR generation uses [node-qrcode](https://github.com/soldair/node-qrcode).

## Transfer settings

The sender and receiver include advanced settings for frame rate, QR capacity, error correction, display size, capture resolution, capture rate, and decoder worker count.

If decoding is slow or unstable, try these values first:

- Reduce **bytes per frame** from `2953` to `1465`.
- Reduce the sender frame rate from `60` to `24` or `30` FPS.
- Increase screen brightness.
- Move the camera closer and keep it steady.

## Privacy and security

Files are processed locally in the browser and are not uploaded to a server. However, the optical channel is **not encrypted**: any camera that can clearly see the sender's screen may capture the transmitted content. Use it as an offline transport mechanism, not as a replacement for encryption.

## Deployment with GitHub Pages

This repository includes a GitHub Actions workflow for Pages deployment. After pushing the project:

1. Open the repository's **Settings**.
2. Select **Pages**.
3. Set the source to **GitHub Actions**.
4. Push to `main` and wait for the deployment workflow to finish.

The project uses relative asset paths, so it can run under a repository subpath such as `username.github.io/decimen-optical-transfer/`.

## Project structure

```text
decimen-optical-transfer/
├── send/       # File and text sender
├── receive/    # Camera capture and QR decoder
├── shared/     # Protocol, fountain code, UI, and shared utilities
├── tests/      # Protocol and unit tests
├── public/     # Static assets
├── build/      # Custom build helpers
└── docs/       # Documentation media
```

## Credits

This repository is a personalized edition of [bashalarmistalt/decimen-optical-transfer](https://github.com/bashalarmistalt/decimen-optical-transfer). The original project and this customized version are distributed under the MIT License.

Customized and maintained with care by **Omid Shojaeian Zanjani**.

## License

Licensed under the [MIT License](LICENSE).
