# Repository Re-architecting Specification: DVSwitch Analog_Bridge
**Target Agent:** Hermes (Repository Reconstruction & Automation Agent)  
**Objective:** Transition the `Analog_Bridge` repository from an in-tree binary storage model to a clean, configuration-first layout that leverages GitHub Releases for binary/`.deb` distribution and maximizes cherry-picking efficiency for users.

---

## 1. Directory Tree Blueprint
Hermes, instantiate or restructure the repository exactly to the following layout. Do not commit any compiled binaries (`.amd64`, `.arm64`, etc.) into the Git tree.

```text
.
├── .github/
│   └── workflows/
│       └── release.yml         # GitHub Actions workflow for building packages & releases
├── config/
│   ├── Analog_Bridge.ini       # Master configuration file template (well-commented)
│   └── dvsm.macro              # Macro configuration template
├── scripts/
│   ├── dvswitch.sh             # Core routing/control shell script
│   └── install_prereqs.sh      # Optional helper to pull system library dependencies
├── src/
│   └── README.md               # Placeholder/documentation for compilation/binary packaging
├── debian/                     # Standard Debian packaging definitions
│   ├── control                 # Package metadata, architecture (any), and dependencies
│   ├── rules                   # Automated instructions for dh_make / dpkg-deb
│   ├── dirs                    # Target system directories to create on installation
│   └── install                 # File-to-destination map for the .deb packager
├── README.md                   # Main documentation highlighting Raw URL cherry-picking
└── LICENSE                     # Project license file
```

---

## 2. Component Specifications

### A. Main Configuration Template (`config/Analog_Bridge.ini`)
Ensure this template is clean, modular, and extensively documented inline for users cherry-picking it via `wget`. 
* Maintain key sections: `[GENERAL]`, `[USRP]`, `[AMBE_AUDIO]`, `[DV_NODE]`.
* Ensure default port mappings are standard (e.g., USRP listening on `31001`, AMBE loop on `31100`).

### B. Debian Packaging Metadata (`debian/control`)
Configure the metadata to support multi-architecture target generation (`amd64`, `arm64`, `armhf`):
```control
Source: analog-bridge
Section: hamradio
Priority: optional
Maintainer: DVSwitch Team <infrastructure@dvswitch.org>
Build-Depends: debhelper-compat (= 13)
Standards-Version: 4.6.2

Package: analog-bridge
Architecture: amd64 arm64 armhf
Depends: ${shlibs:Depends}, ${misc:Depends}, libc6
Description: Audio translation bridge for DVSwitch digital voice networks.
 Converts raw PCM audio streams to/from AMBE/IMBE compressed digital voice frames.
```

---

## 3. GitHub Actions Release Automation (`.github/workflows/release.yml`)
Implement this workflow to automate asset building. When a tag matching `v*` is pushed, Hermes must ensure the workflow pulls the pre-compiled binary artifacts from a secure build staging area (or uses compilation tasks if source code is present), builds the `.deb` architecture-specific packages, and deploys them directly to GitHub Releases.

```yaml
name: Automated Production Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build_packages:
    name: Build & Package Release Assets
    runs-on: ubuntu-latest
    strategy:
      matrix:
        arch: [amd64, arm64]
    
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Environment Variables
        run: echo "VERSION=${GITHUB_REF_NAME#v}" >> $GITHUB_ENV

      - name: Prepare Workspace for Package Building
        run: |
          mkdir -p build_root/usr/local/bin
          mkdir -p build_root/etc/dvswitch
          mkdir -p build_root/lib/systemd/system
          
          # Stage user configurations & scripts
          cp config/Analog_Bridge.ini build_root/etc/dvswitch/Analog_Bridge.ini.example
          cp config/dvsm.macro build_root/etc/dvswitch/dvsm.macro.example
          cp scripts/dvswitch.sh build_root/usr/local/bin/dvswitch.sh
          chmod +x build_root/usr/local/bin/dvswitch.sh

      - name: Fetch Architecture Binary
        run: |
          # Inside a source workflow, compile here. 
          # In an infrastructure-only workflow, fetch the verified pre-compiled core binary:
          # curl -L -o build_root/usr/local/bin/Analog_Bridge https://compiled-storage.dvswitch.org/bin/Analog_Bridge.${{ matrix.arch }}
          # For template generation, creating a verified placeholder:
          echo "Placeholder for Analog_Bridge binary version ${{ env.VERSION }} (${{ matrix.arch }})" > build_root/usr/local/bin/Analog_Bridge
          chmod +x build_root/usr/local/bin/Analog_Bridge

      - name: Build Standalone Tarball Artifact
        run: |
          mkdir -p artifacts
          cp build_root/usr/local/bin/Analog_Bridge artifacts/Analog_Bridge.${{ matrix.arch }}
          cd build_root && tar -czf ../artifacts/analog-bridge-${{ env.VERSION }}-${{ matrix.arch }}.tar.gz .

      - name: Build Debian Package
        run: |
          cp -r debian build_root/DEBIAN
          # Dynamic correction of Architecture field in the control file based on matrix execution
          sed -i 's/^Architecture:.*/Architecture: ${{ matrix.arch }}/' build_root/DEBIAN/control
          sed -i 's/^Version:.*/Version: ${{ env.VERSION }}/' build_root/DEBIAN/control || echo "Version: ${{ env.VERSION }}" >> build_root/DEBIAN/control
          
          dpkg-deb --build build_root artifacts/analog-bridge_${{ env.VERSION }}_${{ matrix.arch }}.deb

      - name: Upload and Attach Assets to GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: |
            artifacts/Analog_Bridge.${{ matrix.arch }}
            artifacts/analog-bridge-${{ env.VERSION }}-${{ matrix.arch }}.tar.gz
            artifacts/analog-bridge_${{ env.VERSION }}_${{ matrix.arch }}.deb
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 4. User-Facing Readme Documentation Template (`README.md`)
Hermes, populate the main `README.md` with explicit instructions on how users can leverage the new repository architecture to pick exactly what they need.

```markdown
# Analog_Bridge
Core audio translation engine for the DVSwitch ecosystem.

## Installation Options

### Option 1: Debian Package Deployment (Recommended)
Go to the [Releases](https://github.com/DVSwitch/Analog_Bridge/releases) tab, select the latest stable release version, and download the `.deb` package matching your system architecture (`amd64`, `arm64`).
```bash
wget https://github.com/DVSwitch/Analog_Bridge/releases/download/v1.6.0/analog-bridge_1.6.0_amd64.deb
sudo apt install ./analog-bridge_1.6.0_amd64.deb
```

### Option 2: Cherry-Picking Specific Configuration Files
If you are updating or repairing an existing installation and only want a specific file without cloning the tree, use the Raw permalinks below:

* **Grab Latest Master Configuration Template (`Analog_Bridge.ini`):**
  ```bash
  wget https://raw.githubusercontent.com/DVSwitch/Analog_Bridge/main/config/Analog_Bridge.ini -O /etc/dvswitch/Analog_Bridge.ini
  ```

* **Grab Master Macro Template (`dvsm.macro`):**
  ```bash
  wget https://raw.githubusercontent.com/DVSwitch/Analog_Bridge/main/config/dvsm.macro -O /etc/dvswitch/dvsm.macro
  ```

* **Grab Latest Router/Control Script (`dvswitch.sh`):**
  ```bash
  wget https://raw.githubusercontent.com/DVSwitch/Analog_Bridge/main/scripts/dvswitch.sh -O /usr/local/bin/dvswitch.sh
  chmod +x /usr/local/bin/dvswitch.sh
  ```
```

---

## 5. Execution Orders for Hermes
1. **Tree Cleanup:** Sweep and remove existing compiled binaries found directly inside the Git tracking branch (`bin/Analog_Bridge.*`). Add any raw `.deb` or binary build extensions to a root `.gitignore`.
2. **Directory Instantiation:** Form the structural directories `/config`, `/scripts`, `/debian`, and `.github/workflows/`.
3. **Template Seeding:** Move or create `Analog_Bridge.ini` and `dvsm.macro` within `/config`. 
4. **Automation Setup:** Commit the `release.yml` GitHub workflow file exactly as specified.
5. **Documentation Commit:** Overwrite the main root `README.md` to highlight the updated raw file cherry-picking paths and Release asset targets.
