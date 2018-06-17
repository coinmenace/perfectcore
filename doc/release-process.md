Release Process
====================

* Update translations (ping wumpus, Diapolo or tcatm on IRC) see [translation_process.md](https://github.com/perfectcoin/perfectcoin/blob/master/doc/translation_process.md#syncing-with-transifex)
* Update [bips.md](bips.md) to account for changes since the last release.
* Update hardcoded [seeds](/contrib/seeds)

* * *

###First time / New builders
Check out the source code in the following directory hierarchy.

	cd /path/to/your/toplevel/build
	git clone https://github.com/perfectcoin/gitian.sigs.git
	git clone https://github.com/perfectcoin/perfectcoin-detached-sigs.git
	git clone https://github.com/devrandom/gitian-builder.git
	git clone https://github.com/perfectcoin/perfectcoin.git

###PerfectCoin maintainers/release engineers, update (commit) version in sources

	pushd ./perfectcoin
	contrib/verifysfbinaries/verify.sh
	configure.ac
	doc/README*
	doc/Doxyfile
	contrib/gitian-descriptors/*.yml
	src/clientversion.h (change CLIENT_VERSION_IS_RELEASE to true)

	# tag version in git

	git tag -s v(new version, e.g. 0.8.0)

	# write release notes. git shortlog helps a lot, for example:

	git shortlog --no-merges v(current version, e.g. 0.7.2)..v(new version, e.g. 0.8.0)
	popd

* * *

###Setup and perform Gitian builds

 Setup Gitian descriptors:

	pushd ./perfectcoin
	export SIGNER=(your Gitian key, ie bluematt, sipa, etc)
	export VERSION=(new version, e.g. 0.8.0)
	git fetch
	git checkout v${VERSION}
	popd

  Ensure your gitian.sigs are up-to-date if you wish to gverify your builds against other Gitian signatures.

	pushd ./gitian.sigs
	git pull
	popd

  Ensure gitian-builder is up-to-date to take advantage of new caching features (`e9741525c` or later is recommended).

	pushd ./gitian-builder
	git pull

###Fetch and create inputs: (first time, or when dependency versions change)

	mkdir -p inputs
	wget -P inputs https://perfectcoincore.org/cfields/osslsigncode-Backports-to-1.7.1.patch
	wget -P inputs http://downloads.sourceforge.net/project/osslsigncode/osslsigncode/osslsigncode-1.7.1.tar.gz

 Register and download the Apple SDK: see [OS X readme](README_osx.txt) for details.

 https://developer.apple.com/devcenter/download.action?path=/Developer_Tools/xcode_6.1.1/xcode_6.1.1.dmg

 Using a Mac, create a tarball for the 10.9 SDK and copy it to the inputs directory:

	tar -C /Volumes/Xcode/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/ -czf MacOSX10.9.sdk.tar.gz MacOSX10.9.sdk

###Optional: Seed the Gitian sources cache and offline git repositories

By default, Gitian will fetch source files as needed. To cache them ahead of time:

	make -C ../perfectcoin/depends download SOURCES_PATH=`pwd`/cache/common

Only missing files will be fetched, so this is safe to re-run for each build.

NOTE: Offline builds must use the --url flag to ensure Gitian fetches only from local URLs. For example:
```
./bin/gbuild --url perfectcoin=/path/to/perfectcoin,signature=/path/to/sigs {rest of arguments}
```
The gbuild invocations below <b>DO NOT DO THIS</b> by default.

###Build and sign PerfectCoin Core for Linux, Windows, and OS X:

	./bin/gbuild --commit perfectcoin=v${VERSION} ../perfectcoin/contrib/gitian-descriptors/gitian-linux.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-linux --destination ../gitian.sigs/ ../perfectcoin/contrib/gitian-descriptors/gitian-linux.yml
    mv build/out/perfectcoin-*.tar.gz build/out/src/perfectcoin-*.tar.gz ../

	./bin/gbuild --commit perfectcoin=v${VERSION} ../perfectcoin/contrib/gitian-descriptors/gitian-win.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-win-unsigned --destination ../gitian.sigs/ ../perfectcoin/contrib/gitian-descriptors/gitian-win.yml
    mv build/out/perfectcoin-*-win-unsigned.tar.gz inputs/perfectcoin-win-unsigned.tar.gz
    mv build/out/perfectcoin-*.zip build/out/perfectcoin-*.exe ../

	./bin/gbuild --commit perfectcoin=v${VERSION} ../perfectcoin/contrib/gitian-descriptors/gitian-osx.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-osx-unsigned --destination ../gitian.sigs/ ../perfectcoin/contrib/gitian-descriptors/gitian-osx.yml
    mv build/out/perfectcoin-*-osx-unsigned.tar.gz inputs/perfectcoin-osx-unsigned.tar.gz
    mv build/out/perfectcoin-*.tar.gz build/out/perfectcoin-*.dmg ../

  Build output expected:

  1. source tarball (perfectcoin-${VERSION}.tar.gz)
  2. linux 32-bit and 64-bit dist tarballs (perfectcoin-${VERSION}-linux[32|64].tar.gz)
  3. windows 32-bit and 64-bit unsigned installers and dist zips (perfectcoin-${VERSION}-win[32|64]-setup-unsigned.exe, perfectcoin-${VERSION}-win[32|64].zip)
  4. OS X unsigned installer and dist tarball (perfectcoin-${VERSION}-osx-unsigned.dmg, perfectcoin-${VERSION}-osx64.tar.gz)
  5. Gitian signatures (in gitian.sigs/${VERSION}-<linux|{win,osx}-unsigned>/(your Gitian key)/

###Verify other gitian builders signatures to your own. (Optional)

  Add other gitian builders keys to your gpg keyring

	gpg --import ../perfectcoin/contrib/gitian-downloader/*.pgp

  Verify the signatures

	./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-linux ../perfectcoin/contrib/gitian-descriptors/gitian-linux.yml
	./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-win-unsigned ../perfectcoin/contrib/gitian-descriptors/gitian-win.yml
	./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-osx-unsigned ../perfectcoin/contrib/gitian-descriptors/gitian-osx.yml

	popd

###Next steps:

Commit your signature to gitian.sigs:

	pushd gitian.sigs
	git add ${VERSION}-linux/${SIGNER}
	git add ${VERSION}-win-unsigned/${SIGNER}
	git add ${VERSION}-osx-unsigned/${SIGNER}
	git commit -a
	git push  # Assuming you can push to the gitian.sigs tree
	popd

  Wait for Windows/OS X detached signatures:
	Once the Windows/OS X builds each have 3 matching signatures, they will be signed with their respective release keys.
	Detached signatures will then be committed to the [perfectcoin-detached-sigs](https://github.com/perfectcoin/perfectcoin-detached-sigs) repository, which can be combined with the unsigned apps to create signed binaries.

  Create (and optionally verify) the signed OS X binary:

	pushd ./gitian-builder
	./bin/gbuild -i --commit signature=v${VERSION} ../perfectcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-osx-signed --destination ../gitian.sigs/ ../perfectcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
	./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-osx-signed ../perfectcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
	mv build/out/perfectcoin-osx-signed.dmg ../perfectcoin-${VERSION}-osx.dmg
	popd

  Create (and optionally verify) the signed Windows binaries:

	pushd ./gitian-builder
	./bin/gbuild -i --commit signature=v${VERSION} ../perfectcoin/contrib/gitian-descriptors/gitian-win-signer.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-win-signed --destination ../gitian.sigs/ ../perfectcoin/contrib/gitian-descriptors/gitian-win-signer.yml
	./bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-win-signed ../perfectcoin/contrib/gitian-descriptors/gitian-win-signer.yml
	mv build/out/perfectcoin-*win64-setup.exe ../perfectcoin-${VERSION}-win64-setup.exe
	mv build/out/perfectcoin-*win32-setup.exe ../perfectcoin-${VERSION}-win32-setup.exe
	popd

Commit your signature for the signed OS X/Windows binaries:

	pushd gitian.sigs
	git add ${VERSION}-osx-signed/${SIGNER}
	git add ${VERSION}-win-signed/${SIGNER}
	git commit -a
	git push  # Assuming you can push to the gitian.sigs tree
	popd

-------------------------------------------------------------------------

### After 3 or more people have gitian-built and their results match:

- Create `SHA256SUMS.asc` for the builds, and GPG-sign it:
```bash
sha256sum * > SHA256SUMS
gpg --digest-algo sha256 --clearsign SHA256SUMS # outputs SHA256SUMS.asc
rm SHA256SUMS
```
(the digest algorithm is forced to sha256 to avoid confusion of the `Hash:` header that GPG adds with the SHA256 used for the files)
Note: check that SHA256SUMS itself doesn't end up in SHA256SUMS, which is a spurious/nonsensical entry.

- Upload zips and installers, as well as `SHA256SUMS.asc` from last step, to the perfectcoin.org server
  into `/var/www/bin/perfectcoin-core-${VERSION}`

- Update perfectcoin.org version

  - First, check to see if the PerfectCoin.org maintainers have prepared a
    release: https://github.com/perfectcoin-dot-org/perfectcoin.org/labels/Releases

      - If they have, it will have previously failed their Travis CI
        checks because the final release files weren't uploaded.
        Trigger a Travis CI rebuild---if it passes, merge.

  - If they have not prepared a release, follow the PerfectCoin.org release
    instructions: https://github.com/perfectcoin-dot-org/perfectcoin.org#release-notes

  - After the pull request is merged, the website will automatically show the newest version within 15 minutes, as well
    as update the OS download links. Ping @saivann/@harding (saivann/harding on Freenode) in case anything goes wrong

- Announce the release:

  - Release sticky on perfectcointalk: https://perfectcointalk.org/index.php?board=1.0

  - PerfectCoin-development mailing list

  - Update title of #perfectcoin on Freenode IRC

  - Optionally reddit /r/PerfectCoin, ... but this will usually sort out itself

- Notify BlueMatt so that he can start building [the PPAs](https://launchpad.net/~perfectcoin/+archive/ubuntu/perfectcoin)

- Add release notes for the new version to the directory `doc/release-notes` in git master

- Celebrate
