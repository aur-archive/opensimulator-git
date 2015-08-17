# Contributor: Anton Bazhenov <anton.bazhenov at gmail>
# Contributor: LTSmash <lord.ltsmash@gmail.com>
# Contributor: Zauber Exonar <zauberexonar at gmail>
# Contributor: Simon Peter Nicholls <simon.peter.nicholls at googlemail>
# Contributor: GordonGR <ntheo1979@gmail.com>
# Maintainer: Thonneve <nathanmarck@live.com>

pkgname=opensimulator-git
pkgver=0.8.3a7c8d1f32
pkgrel=1
pkgdesc="A 3D application server used to create a virtual environment or world"
arch=('i686' 'x86_64')
url="http://opensimulator.org/wiki/Main_Page"
license=('BSD')
depends=('mono' 'sqlite')
optdepends=('mysql')
install=$pkgname.install
source=("git://opensimulator.org/git/opensim"
		"opensimulator.sh"
		"opensimulator-robust.sh"
		"opensimulator.service"
		"opensimulator-robust.service")
md5sums=('SKIP'
         '163937630992d7dcb7c6ad05c7424fe1'
         '1b9e4f0c39e314caba59486a12ac1858'
         'dbce38d9473d3ebcd4f232c0fd0f90ab'
         'caa8050132aecb8face9780b43038cd9')
backup=(opt/$pkgname/bin/OpenSim.ini)

build() {
  cd "$srcdir"/opensim

  # we need Mono
  export MONO_SHARED_DIR="$srcdir"/.wabi
  mkdir -p $MONO_SHARED_DIR

  # build opensimulator using nant
  ./runprebuild.sh
  xbuild
}

package() {
  cd "$srcdir"/opensim

  # delete unneeded and create log/ini files
  [[ `uname -m` = "i686" ]] && find bin -name "*x86_64.so" -delete

  # ensure log file already exists
  touch bin/OpenSim.log

  # create a default OpenSim.ini for installs
  sed 's/^\(\s*\)\; \(Include.*Standalone\.ini\)/\1\2/' bin/OpenSim.ini.example >bin/OpenSim.ini

  #copying Mono.Posix.dll so that OpenSimulator can use sockets, and by extension MySQL
  cp /usr/lib/mono/2.0/Mono.Posix.dll bin/Mono.Posix.dll

  # install
  install -d "$pkgdir"/opt/$pkgname/bin
  cp -r bin/* "$pkgdir"/opt/$pkgname/bin/
  mv README.md README.txt
  install -Dm644 {CONTRIBUTORS,README}.txt "$pkgdir"/opt/$pkgname

  # set permissions
  find "$pkgdir"/opt/$pkgname/bin -type d -exec chmod 755 {} +
  find "$pkgdir"/opt/$pkgname/bin -type f -exec chmod 644 {} +
  find "$pkgdir"/opt/$pkgname/bin -name "*.exe" -exec chmod 755 {} +
  find "$pkgdir"/opt/$pkgname/bin -name "*.ini" -exec chmod 666 {} +
  find "$pkgdir"/opt/$pkgname/bin -name "*.xml" -exec chmod 666 {} +
  chmod 777 "$pkgdir"/opt/$pkgname/bin/{,*/}
  chmod 755 "$pkgdir"/opt/$pkgname/bin/opensim-ode.sh
  chmod 666 "$pkgdir"/opt/$pkgname/bin/OpenSim.log

  # install scripts, service and license files
  install -m755 -D ../opensimulator.sh "$pkgdir"/usr/bin/$pkgname
  install -m755 -D ../opensimulator-robust.sh "$pkgdir"/usr/bin/opensimulator-robust-git
  install -Dm644 "$srcdir/opensimulator.service" "$pkgdir/usr/lib/systemd/system/opensimulator-git@.service"
  install -Dm644 "$srcdir/opensimulator-robust.service" "$pkgdir/usr/lib/systemd/system/opensimulator-robust-git@.service"
  install -m644 -D LICENSE.txt "$pkgdir"/usr/share/licenses/$pkgname/LICENSE
  
  # remove *.dylib files (thanks to Schala)
  cd "$pkgdir"/opt/$pkgname/bin
  rm -f libopenjpeg-dotnet-2-1.5.0-dotnet-1.dylib
  rm -f lib32/libBulletSim.dyliblib64/libode.dylib
  rm -rf lib64/{libopenjpeg-dotnet.dylib,libsqlite3.dylib}
}
