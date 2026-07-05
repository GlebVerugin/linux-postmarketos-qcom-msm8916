# Maintainer: Verugin Gleb <gleb@glebmail.xyz>
# Kernel config based on: arch/arm64/configs/msm8916_defconfig

# Original authors Minecrell <minecrell@minecrell.net> and Nikita Travkin <nikita@trvn.ru>, based on pmos package

_flavor="postmarketos-qcom-msm8916"
pkgname=linux-$_flavor
pkgver=6.12.95
pkgrel=1
pkgdesc="Mainline kernel fork for Qualcomm MSM8909/MSM8916/MSM8939 devices"
arch="aarch64 armv7"
url="https://github.com/GlebVerugin/msm8916-linux"
license="GPL-2.0-only"
options="!strip !check !tracedeps
	pmb:cross-native
	pmb:kconfigcheck-community
	pmb:kconfigcheck-uefi
	pmb:kconfigcheck-usb
"
makedepends="
	zstd
	bison
	findutils
	flex
	gmp-dev
	mpc1-dev
	mpfr-dev
	openssl-dev
	perl
	postmarketos-installkernel
	python3
	rsync
"
replaces="linux-postmarketos-qcom-msm8909 linux-postmarketos-qcom-msm8939"

# Architecture
case "$CARCH" in
	aarch64) _carch="arm64" ;;
	arm*)    _carch="arm" ;;
esac

# Source
_tag=v${pkgver//_/-}-msm8916
source="
	$pkgname-$_tag.tar.gz::$url/archive/$_tag.tar.gz
	config-$_flavor.aarch64
	config-$_flavor.armv7
"
builddir="$srcdir/msm8916-linux-${_tag#v}"

prepare() {
	default_prepare
	cp "$srcdir/config-$_flavor.$CARCH" .config
}

build() {
	unset LDFLAGS
	make -j$(nproc) ARCH="$_carch" CC="${CC:-gcc}" \
		KBUILD_BUILD_VERSION=$((pkgrel + 1 ))
    make headers
}

package() {
	mkdir -p "$pkgdir"/boot

	_install_targets="modules_install dtbs_install headers_install"

	if [ -e "$builddir/arch/$_carch/boot/vmlinuz.efi" ]; then
		# ZBOOT EFI decompressor for EFI booting
		install -Dm644 "$builddir/arch/$_carch/boot/vmlinuz.efi" \
			"$pkgdir/boot/linux.efi"

		# Old GZIP'd kernel image for boot.img compatibility
		install -Dm644 "$builddir/arch/$_carch/boot/vmlinuz" \
			"$pkgdir/boot/vmlinuz"
	elif [ "$_carch" = "arm64" ]; then
		echo "WARNING: CONFIG_ZBOOT not enabled!"
		install -Dm644 "$builddir/arch/$_carch/boot/Image.gz" \
			"$pkgdir/boot/vmlinuz"
	else
		_install_targets="zinstall modules_install dtbs_install headers_install"
	fi

	make $_install_targets \
		ARCH="$_carch" \
		INSTALL_PATH="$pkgdir"/boot \
		INSTALL_MOD_PATH="$pkgdir" \
		INSTALL_MOD_STRIP=1 \
		INSTALL_DTBS_PATH="$pkgdir"/boot/dtbs \
        INSTALL_HDR_PATH="$pkgdir/usr"
	rm -f "$pkgdir"/lib/modules/*/build "$pkgdir"/lib/modules/*/source

	install -D "$builddir"/include/config/kernel.release \
		"$pkgdir"/usr/share/kernel/$_flavor/kernel.release
}

sha512sums="
2c7f819238db7b9195f37752edbfe2f7a9db1f13dce8c74d44ac8c5b8e3633bac432179a56fb2347fbb906eb7d2e6faa4630f9b64bec125254c8ed089ce7a98b  linux-postmarketos-qcom-msm8916-v6.12.95-msm8916.tar.gz
36f45a9765257d994eb1aa562c58683ef593dc0d02448221ceda4b3fb7f0f2327e5ab08d0075ee708ddb3c2087e1af0b502770e5be48c304c6aa6f6ecec666c5  config-postmarketos-qcom-msm8916.aarch64
c76edf1f01a98d73e42bf01fe980dcfe0752e51946c98027314d336e9fc08b513a4bb8dfcde1b29467e661c77ffeb6b7d0e362b83efeebc3c81accd649e38d39  config-postmarketos-qcom-msm8916.armv7
"
