# Setup
import os
import sys
env = Environment(ENV = os.environ)
try:
    env.Tool('config', toolpath = [os.environ.get('CBANG_HOME')])
except Exception, e:
    raise Exception, 'CBANG_HOME not set?\n' + str(e)

env.CBLoadTools('packager fah-client-version')
conf = env.CBConfigure()

# Version
version = env.FAHClientVersion()

# this should be in packager.configure
env.Append(PACKAGE_IGNORES = ['.DS_Store'])

# Sub Packages
for v in 'FAH_VIEWER FAH_CLIENT FAH_CONTROL FAH_CLIENT_OSX_UNINSTALLER'.split():
    if not v + '_HOME' in os.environ: raise Exception, '%s_HOME not set' % v

un_home = os.environ.get('FAH_CLIENT_OSX_UNINSTALLER_HOME')
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
    'distpkg_background' : 'fah-light.png',
    'distpkg_target' : '10.6',
    'distpkg_arch' : env.get('package_arch', 'x86_64'),
    'distpkg_flat' : True,
    'distpkg_components' : distpkg_components,
    }
pkg = env.Packager(**parameters)

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
        ])
