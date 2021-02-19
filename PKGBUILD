# Maintainer: Fredy García <frealgagu at gmail dot com>
# Contributor: klepz <felipe.junger@aluno.ufabc.edu.br>
# Contributor: Dave Blair <mail@dave-blair.de>
# Contributor: James An <james@jamesan.ca>
# Contributor: Mateus Rodrigues Costa <charles [dot] costar [at] gmail [dot] com>

pkgname=chrome-remote-desktop-patched
pkgver=88.0.4324.34
pkgrel=1
pkgdesc="Access other computers or allow another user to access your computer securely over the Internet"
arch=("x86_64")
url="https://remotedesktop.google.com"
license=("BSD")
depends=("gtk3" "libxss" "nss" "python-psutil" "xorg-server-xvfb" "xorg-setxkbmap" "xorg-xauth" "xorg-xdpyinfo" "xorg-xrandr")
install="${pkgname:0:21}.install"
source=(
  "${pkgname:0:21}-${pkgver}.deb::https://dl.google.com/linux/${pkgname:0:21}/deb/pool/main/${pkgname:0:1}/${pkgname:0:21}/${pkgname:0:21}_${pkgver}_amd64.deb"
  "${pkgname:0:21}.service"
  "pamrule"
  "crd"
)
sha256sums=(
  "bfb9568d5eab477bfd7816a5281910a8709b2217eb926144eb0e759e63af6be0"
  "e5da5ae89b5bc599f72f415d1523341b25357931b0de46159fce50ab83615a4b"
  "fcc38269eb1cc902abff9688eda9377a22367e39b9f111f87c0dd8e77adb82e2"
  "27dee2d383e6bd993fe0557d5c222fa80ab6d16d43775dedff6218713c7a1c06"
)

# curl -qs https://dl.google.com/linux/chrome-remote-desktop/deb/dists/stable/main/binary-amd64/Packages | grep "^Version\|^SHA256" | awk '{print $2}'

build() {
  cd "${srcdir}"

  bsdtar -xf data.tar.xz -C .

  # Patch for duplicating existing session
  sed -i -e 's#FIRST_X_DISPLAY_NUMBER = 20#FIRST_X_DISPLAY_NUMBER = 20\nEXISTING_X_DISPLAY_FILE_PATH = os.path.join(CONFIG_DIR, "Xsession")\nX_SESSION_FILE_TEMPLATE = "/tmp/.X11-unix/X%d"#g' opt/google/chrome-remote-desktop/chrome-remote-desktop
sed -i -e 's#raise Exception("Could not start X session")#raise Exception("Could not start X session")\n\n  def _use_existing_session(self):\n    with open(EXISTING_X_DISPLAY_FILE_PATH) as fh:\n      display = int(fh.readline().rstrip())\n    if not os.path.exists(X_SESSION_FILE_TEMPLATE % display):\n      logging.error("Xorg session file doesn t exist")\n      sys.exit(1)\n    logging.info("Using existing Xorg session: %d" % display)\n    self.child_env["DISPLAY"] = ":%d" % display\n    self.child_env["CHROME_REMOTE_DESKTOP_SESSION"] = "1"\n#g' opt/google/chrome-remote-desktop/chrome-remote-desktop
sed -i -e 's#    self._launch_x_server(x_args)#    if os.path.exists(EXISTING_X_DISPLAY_FILE_PATH):\n      self._use_existing_session()\n    else:\n      self._launch_x_server(x_args)\n #g' opt/google/chrome-remote-desktop/chrome-remote-desktop
sed -i -e 's#    self._launch_x_session()#      self._launch_x_session()#g' opt/google/chrome-remote-desktop/chrome-remote-desktop
  
  # Removing unnecessary .deb related files
  rm -R "${srcdir}/etc/cron.daily"
  rm -R "${srcdir}/etc/init.d"
  rm -R "${srcdir}/etc/pam.d"
}

package() {
  cd "${srcdir}"

  install -d "${pkgdir}/etc"
  install -d "${pkgdir}/opt"
  cp -r "${srcdir}/etc/"* "${pkgdir}/etc"
  cp -r "${srcdir}/opt/"* "${pkgdir}/opt"
  install -Dm644 "${srcdir}/usr/share/doc/${pkgname:0:21}/copyright" "${pkgdir}/usr/share/licenses/${pkgname:0:21}/copyright"
  install -Dm644 "${srcdir}/${pkgname:0:21}.service" "${pkgdir}/usr/lib/systemd/user/${pkgname}.service"
  install -Dm644 "${srcdir}/pamrule" "${pkgdir}/etc/pam.d/${pkgname:0:21}"
  install -Dm755 "${srcdir}/crd" "${pkgdir}/usr/bin/crd"
  install -dm755 "${pkgdir}/etc/chromium/native-messaging-hosts"
  
  for _file in $(find "${pkgdir}/etc/opt/chrome/native-messaging-hosts" -type f); do
    local _filename=$(basename ${_file})
    if [[ ! -f "/etc/chromium/native-messaging-hosts/${_filename}" ]]; then
      ln -s "/etc/opt/chrome/native-messaging-hosts/${_filename}" "${pkgdir}/etc/chromium/native-messaging-hosts/${_filename}"
    fi
  done
  chmod u+s "${pkgdir}/opt/google/${pkgname:0:21}/user-session"
}
