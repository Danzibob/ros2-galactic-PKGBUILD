# Maintainer: Mohammad Mostafa Farzan <m2_farzan@yahoo.com>
# Contributor: mjbogusz <mjbogusz+github@gmail.com>
# Acknowledgment: This work is hugely based on `ros2-arch-deps` AUR
# package, maintained by T. Borgert.

pkgname=ros2-galactic
pkgver=2021.05.23
pkgrel=3
pkgdesc="A set of software libraries and tools for building robot applications"
url="https://docs.ros.org/en/galactic/"
arch=('any')
license=('Apache')
depends=(
	'ros2-arch-deps'
	'gmock'
	'sip4'
)
source=(
	"ros2::git+https://github.com/ros2/ros2#tag=release-galactic-20210523"
	"google_benchmark_vendor.patch"
)
sha256sums=(
	'SKIP'
	"609a5260736192608582c0f0a0fd4da09a9185d95d452a92d9527af38d720f6a"
)
install=ros2-galactic.install

prepare() {
	# Check locale
	locale | grep LANG | grep UTF-8
	if [[ $? -ne 0 ]]; then
		printf "Locale must support UTF-8. See https://wiki.archlinux.org/index.php/locale
		or https://wiki.archlinux.org/index.php/locale ."
		exit 1
	fi

	# Clone the repos
	mkdir -p $srcdir/ros2/src
	vcs import $srcdir/ros2/src < $srcdir/ros2/ros2.repos

	# Fix some issues in the code (TODO: Gradually move to upstream)
	## google_benchmark_vendor
	git -C $srcdir/ros2/src/ament/google_benchmark_vendor checkout .
	git -C $srcdir/ros2/src/ament/google_benchmark_vendor apply $srcdir/google_benchmark_vendor.patch
	## Eclipse iceoryx
	git -C $srcdir/ros2/src/eclipse-iceoryx/iceoryx checkout release_1.0
	## Eclipse CycloneDDS
	git -C $srcdir/ros2/src/eclipse-cyclonedds/cyclonedds checkout 0.8.0beta6
	git -C $srcdir/ros2/src/eclipse-cyclonedds/cyclonedds cherry-pick bdf270a588aae77d0f1a0f0070b53ad1388da61c
}

build() {
	# Disable parallel build if RAM is low
	if [[ $(free | grep -Po "Mem:\s+\K\d+") < 16000000 ]]; then
		printf "\nRAM is smaller than 16 GB. Parallel build will be disabled for stability.\n\n"
		export COLCON_EXTRA_ARGS="${COLCON_EXTRA_ARGS} --executor sequential"
	fi

	# Disable -D_FORTIFY_SOURCE=2 flag (Required to build mimick_vendor)
	## For people with the old version of makepkg.conf
	unset CPPFLAGS
	## For people with the new version of makepkg.conf
	CFLAGS=$(sed "s/-Wp,-D_FORTIFY_SOURCE=2\s//g" <(echo $CFLAGS))
	CXXFLAGS=$(sed "s/-Wp,-D_FORTIFY_SOURCE=2\s//g" <(echo $CXXFLAGS))

	# Build
	colcon build --merge-install ${COLCON_EXTRA_ARGS}
}

package() {
	mkdir -p $pkgdir/opt/ros2/galactic
	cp -r $srcdir/install/* $pkgdir/opt/ros2/galactic/
}
