# Setup
import os
import sys
env = Environment()
try:
    env.Tool('config', toolpath = [os.environ.get('CBANG_HOME')])
except Exception, e:
    raise Exception, 'CBANG_HOME not set?\n' + str(e)

env.CBAddVariables(
    # desire everything built flat True, target 10.5
    # to get old build, use flat False, target 10.6 in scons-options.py
    BoolVariable('distpkg_flat', 'Build a flat OSX installer pkg', True),
    ('distpkg_target', 'Min OSX version required by installer pkg', '10.5'),
    # non-root pkg installs are very buggy, so this should stay True
    ('distpkg_root_volume_only', 'Require root auth to install', True),
    # put sign_* in scons-options.py
    # if not sign_keychain, the default (login) keychain will be used
    # if not sign_id_installer, productsign will be skipped
    # sign_id_app is required for sign_apps and sign_tools
    # sign_prefix is required for sign_tools
    # global; cannot currently be overridden per-component
    ('sign_keychain', 'Keychain that has signatures'),
    ('sign_id_installer', 'Installer signature name'),
    ('sign_id_app', 'Application/Tool signature name'),
    ('sign_prefix', 'codesign identifier prefix'))

env.CBLoadTools('packager')
conf = env.CBConfigure()

# Version
version = open('version/version.txt', 'r').read().strip()
env.Replace(PACKAGE_VERSION = version)

# this should be in packager.configure
env.Append(PACKAGE_IGNORES = ['.DS_Store'])

sys.path.append('./src')
import flatdistpkg, flatdistpackager
flatdistpkg.configure(conf)
flatdistpackager.configure(conf)

# Sub Packages
for v in 'FAH_VIEWER FAH_CLIENT FAH_CONTROL FAH_CLIENT_OSX_UNINSTALL'.split():
    if not v + '_HOME' in os.environ: raise Exception, '%s_HOME not set' % v

# needed only for old non-flat build
packages = (
    os.environ.get('FAH_CLIENT_HOME') + '/FAHClient.pkg',
    os.environ.get('FAH_VIEWER_HOME') + '/FAHViewer.pkg',
    os.environ.get('FAH_CONTROL_HOME') + '/FAHControl.pkg',
)

un_home = os.environ.get('OSX_UNINSTALL_HOME')
un_root = './dist/flatpkg/Uninstaller/root'
un_pkg_name_file = os.path.join(un_home, 'package.txt')
if not env.GetOption('clean'):
    un_pkg_name = open(un_pkg_name_file).read().strip()
else:
    un_pkg_name = ''
# replace '.mpkg.zip' with '.pkg'
un_pkg_name = os.path.splitext(os.path.splitext(un_pkg_name)[0])[0] + '.pkg'
un_pkg_files = [[os.path.join(un_home, un_pkg_name),
            'Applications/Folding@home/Uninstall Folding@home.pkg']]

if not env.GetOption('clean'):
    # create dist root for our uninstaller component
    import shutil
    if os.path.exists(un_root): shutil.rmtree(un_root)
    env.CopyToPackage(un_pkg_files, un_root)

# Flat Dist Components
# We must build component pkgs from scratch, but require each component
# distroot to have been already created for us, and assume known layout
# used by pkg.py and current projects.
# So, we currently expect pre-existence of (relative to component home)
#   build/pkg/root (or set 'root' in components)
#   build/pkg/Resources  (optional; or set 'resources' in components)
#   osx/Resources (optional fallback, if no resources or pkg_resources)
#   osx/scripts (optional; or set 'pkg_scripts' in components)
#   product-description.txt (optional, used for choice description)
# Most keys are meant to be the same as passed to Packager in projects.
# Value for home should be an absolute path. Relative to here might work.
# Most other path values are relative to component home.
# Required keys are name, home, pkg_id.
distpkg_components = [
    {'name': 'FAHClient',
        'home': os.environ.get('FAH_CLIENT_HOME'),
        'pkg_id': 'edu.stanford.folding.fahclient.pkg',
        # the cores that are run by the client currently require OSX 10.6+
        # so disable component install for pre-10.6
        # we don't use pkg_target, because that might be 10.4 for bundle build
        # (which becomes packagemaker --target 10.4)
        # REMOVE this once all FahCores run on 10.5
        'distpkg_target': '10.6',
        # must-close seems to require 10.7+
        # close anything that possibly launched or connected to fahclient
        # list of CFBundleIdentifier
        'must_close_apps': [
            'edu.stanford.folding.fahviewer',
            'edu.stanford.folding.fahcontrol',
            ],
        # paths relative to 'root'
        'sign_tools': ['usr/bin/FAHClient', 'usr/bin/FAHCoreWrapper'],
        },
    {'name': 'FAHViewer',
        'home': os.environ.get('FAH_VIEWER_HOME'),
        'pkg_id': 'edu.stanford.folding.fahviewer.pkg',
        'must_close_apps': ['edu.stanford.folding.fahviewer'],
        'sign_apps': ['Applications/Folding@home/FAHViewer.app'],
        },
    {'name': 'FAHControl',
        'home': os.environ.get('FAH_CONTROL_HOME'),
        'pkg_id': 'edu.stanford.folding.fahcontrol.pkg',
        'must_close_apps': ['edu.stanford.folding.fahcontrol'],
        'sign_apps': ['Applications/Folding@home/FAHControl.app'],
        },
    {'name': 'Uninstaller',
        'home': un_home,
        'pkg_id': 'edu.stanford.folding.uninstaller.pkg',
        'root': un_root,
        'pkg_files': un_pkg_files,
        },
    ]

distpkg_target = env.get('distpkg_target')
distpkg_flat = env.get('distpkg_flat')

# special case for fah-installer
# old non-flat DistPkg must target 10.6+ because it can't
# disable fahclient.pkg for 10.5 or earlier
# REMOVE this once all FahCores run on 10.5, or if dropping non-flat support
if not distpkg_flat and distpkg_target.split('.') < (10,6):
    print 'WARNING: changing distpkg_target %s to 10.6 for non-flat pkg' \
        % distpkg_target
    distpkg_target = '10.6'

# note trailing dot; should be set in scons-options.py
sign_prefix = env.get('sign_prefix', 'edu.stanford.folding.')

# Package
name = 'fah-installer'
parameters = {
    'name' : name,
    'version' : version,
    'maintainer' : 'Joseph Coffland <joseph@cauldrondevelopment.com>',
    'vendor' : 'Folding@home',
    'summary' : 'Folding@home ' + version,
    'description' : 'Folding@home ' + version + ' Installer Package',
    'pkg_type' : 'dist',
    'distpkg_resources' : [['Resources', '.']],
    'distpkg_welcome' : 'Welcome.rtf',
    'distpkg_license' : 'License.rtf',
    'distpkg_conclusion' : 'Conclusion.rtf',
    'distpkg_background' : 'fah-light.png',
    'distpkg_target' : distpkg_target,
    'distpkg_arch' : env.get('package_arch', 'x86_64'),
    'distpkg_flat' : distpkg_flat,
    'distpkg_components' : distpkg_components,
    'distpkg_packages' : packages, # only needed if building old non-flat
    'sign_prefix' : sign_prefix,
    }
pkg = env.FlatDistPackager(**parameters)

AlwaysBuild(pkg)
env.Alias('package', pkg)

# Clean
Clean(pkg, ['build', 'dist', 'config.log'])
# ensure *.zip not cleaned unless distclean
NoClean(pkg, Glob('*.zip'))
if 'distclean' in COMMAND_LINE_TARGETS:
    Clean('distclean', [
        '.sconsign.dblite', '.sconf_temp', 'config.log',
        'build', 'dist', 'package.txt', 'package-description.txt',
        Glob(name + '*.pkg'),
        Glob(name + '*.mpkg'),
        Glob(name + '*.zip'),
        Glob('*.pyc'),
        Glob('src/*.pyc'),
        ])
