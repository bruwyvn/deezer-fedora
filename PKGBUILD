# Maintainer: Sibren Vasse <arch@sibrenvasse.nl>
# Contributor: Ilya Gulya <ilyagulya@gmail.com>
pkgname="deezer"
pkgver=4.18.0
pkgrel=1
pkgdesc="A proprietary music streaming service"
arch=('any')
url="https://www.deezer.com/"
license=('custom:"Copyright (c) 2006-2018 Deezer S.A."')
depends=('electron6')
provides=('deezer')
makedepends=('p7zip' 'asar' 'prettier' 'imagemagick' 'npm')
source=("$pkgname-$pkgver-setup.exe::https://www.deezer.com/desktop/download/artifact/win32/x86/$pkgver"
        "$pkgname.desktop"
        systray.patch
        menu-bar.patch
        oauth.patch
        0001-MPRIS-interface.patch)
sha256sums=('6af0d33489a82399a814928daa96cece5eaf4b3276b7ede2bf88c1c78a3b160d'
            'f8a5279239b56082a5c85487b0c261fb332623f27dac3ec8093458b8c55d8d99'
            'f7df7ba3dd91d8d327c3bfe69c65d3969b3f7a80ca253c29bad316e09f65ab8b'
            '8a22f666e308663cb6addabe7695b1e5e3bfa07f68cc7b479e51426dee1c36b0'
            '5cbe1696d5f948cdda1fdd5ef9f8d94d5e14d8594558e35472517fa659057add'
            '217d899797908004453e9c0d86057b5682b3612c6412b4f6c107ac4ad201320b'
            '8a8a42fd38c6fc5a5f9523620ce7e794355ceec5d71c93a7cee378c9a5b3d8ec')

prepare() {
    # Extract app from installer
    7z x -so $pkgname-$pkgver-setup.exe "\$PLUGINSDIR/app-32.7z" > app-32.7z
    # Extract app archive
    7z x -y -bsp0 -bso0 app-32.7z

    # Extract png from ico container
    convert resources/win/app.ico resources/win/deezer.png

    cd resources/
    rm -r app "$srcdir/npm_temp" || true
    asar extract app.asar app

    # Remove NodeRT from package (-205.72 MiB)
    rm -r app/node_modules/@nodert

    # Install extra node modules for mpris-service
    mkdir "$srcdir/npm_temp"; cd "$srcdir/npm_temp"
    npm install  --prefix ./ mpris-service

    for d in node_modules/*; do
        if [ ! -d "$srcdir/resources/app/node_modules/$(basename $d)" ]
        then
            cp -r "$d" "$srcdir/resources/app/node_modules/"
        fi
    done

    cd "$srcdir/resources/app"

    prettier --write "build/*.js"
    # Fix crash on startup since 4.14.1 (patch systray icon path)
    patch -p1 < "$srcdir/systray.patch"
    # Disable menu bar
    patch -p1 < "$srcdir/menu-bar.patch"

    # Monkeypatch MPRIS D-Bus interface
    patch -p1 < "$srcdir/0001-MPRIS-interface.patch"

    cd ..
    asar pack app app.asar
}

package() {
    mkdir -p "$pkgdir"/usr/share/deezer
    mkdir -p "$pkgdir"/usr/share/applications
    mkdir -p "$pkgdir"/usr/share/icons/hicolor/256x256/apps/
    mkdir -p "$pkgdir"/usr/bin/

    echo "#!/bin/sh" > deezer
    echo "exec electron6 /usr/share/deezer/app.asar \"\$@\"" >> deezer

    install -Dm644 resources/app.asar "$pkgdir"/usr/share/deezer/
    install -Dm644 resources/win/deezer.png "$pkgdir"/usr/share/icons/hicolor/256x256/apps/
    install -Dm644 resources/win/systray.png "$pkgdir"/usr/share/deezer/
    install -Dm644 "$pkgname".desktop "$pkgdir"/usr/share/applications/
    install -Dm755 deezer "$pkgdir"/usr/bin/
}
