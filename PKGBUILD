# Maintainer: Nikolay Bryskin <nbryskin@gmail.com>
# Contributor: Pellegrino Prevete <pellegrinoprevete@gmail.com>

# shellcheck disable=SC2034
_pkgbase=metamask
pkgbase=${_pkgbase}-latest
_pkgname="${_pkgbase}-extension"
pkgname=("${pkgbase}-chromium" "${pkgbase}-firefox")
_pkgver=11.16.15
pkgver=11.16.15
pkgrel=2
pkgdesc='Chrome extension that enables browsing Ethereum blockchain enabled websites'
url="https://github.com/MetaMask/${_pkgname}"
license=('custom:consensys')
groups=('firefox-addons')
arch=('any')
depends=('chromium' 'typescript')
makedepends=('git' 'nvm' 'yarn-berry' 'chromium')
source=(
  "${pkgbase}::git+$url.git#tag=v${_pkgver}"
  "chrome.pem"
)
sha512sums=(
  'SKIP'
  '6732e47d2431484e084512e815e44590c6693d9a1f7a192c0de2f11561fdfd6d87769a375650c4025ab8e2049fd7694ff2c45e19a03991d65c9b8a241bd7292a'
)
chromium_extension_id="cfcbhkcbidaoaeljekeilbnebipmnkjm"

_ensure_local_nvm() {
    # let's be sure we are starting clean
    which nvm >/dev/null 2>&1 && nvm deactivate && nvm unload
    export NVM_DIR="${srcdir}/.nvm"

    # The init script returns 3 if version specified
    # in ./.nvrc is not (yet) installed in $NVM_DIR
    # but nvm itself still gets loaded ok
    source /usr/share/nvm/init-nvm.sh || [[ $? != 1 ]]
}

prepare() {
  cd "${srcdir}/${pkgbase}"
  cp .metamaskrc.dist .metamaskrc
  # set infura project id
  sed -i -e 's/00000000000/2f8ebfee0f81453d83fe6219b9a59754/g' .metamaskrc
  # nodejs 20.15 fails with:
  # TypeError: Cannot read properties of undefined (reading '0')
  _ensure_local_nvm
  nvm install 20.14
}

build() {
  cd "${srcdir}/${pkgbase}"
  _ensure_local_nvm
  yarn # setup
  yarn dist:mv2
}

package_metamask-latest-chromium() {
  conflicts=(metamask-chromium)
  replaces=(metamask-chromium)
  replaces=(provides-chromium)
  chromium \
    --disable-gpu \
    --disable-namespace-sandbox \
    --pack-extension="${srcdir}/${pkgbase}/dist/chrome" \
    --pack-extension-key="${srcdir}/chrome.pem"
  crx_path="/usr/lib/chromium-extension-metamask/metamask-${_pkgver}.crx"
  install -Dm644 "${srcdir}/${pkgbase}/dist/chrome.crx" "${pkgdir}${crx_path}"
  extensions_dir="${pkgdir}/usr/share/chromium/extensions"
  cat > "${srcdir}/${chromium_extension_id}.json" <<EOF
{
	"external_crx": "${crx_path}",
	"external_version": "${_pkgver}"
}
EOF
  install -Dm644 "${srcdir}/${chromium_extension_id}.json" "${extensions_dir}/${chromium_extension_id}.json"
}

package_metamask-latest-firefox() {
  conflicts=(metamask-firefox)
  replaces=(metamask-firefox)
  replaces=(provides-firefox)
  extensions_dir="${pkgdir}/usr/lib/firefox/browser/extensions"
  install -Dm644 "${srcdir}/${pkgbase}/builds/metamask-firefox-${_pkgver}.zip" \
                 "${extensions_dir}/webextension@metamask.io.xpi"
}
