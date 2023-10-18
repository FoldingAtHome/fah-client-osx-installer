import os
import sys
env = Environment(ENV = os.environ)
try:
    env.Tool('config', toolpath = [os.environ.get('CBANG_HOME')])
except Exception as e:
    raise Exception('CBANG_HOME not set?\n' + str(e))

env.CBLoadTools('packager fah-client-version')
conf = env.CBConfigure()

version = env.FAHClientVersion()

def eprint(*args, **kwargs):
    print(*args, file=sys.stderr, **kwargs)

# Sub Packages
for v in 'FAH_VIEWER FAH_CLIENT FAH_CONTROL FAH_CLIENT_OSX_UNINSTALLER'.split():
    if not v + '_HOME' in os.environ: raise Exception('%s_HOME not set' % v)

fah_control_home = os.environ.get('FAH_CONTROL_HOME')
fah_control_prebuilt_home = os.environ.get('FAH_CONTROL_PREBUILT_HOME')

un_home = os.environ.get('FAH_CLIENT_OSX_UNINSTALLER_HOME')
un_root = './dist/flatpkg/Uninstaller/root'
un_pkg_name_file = os.path.join(un_home, 'package.txt')
if not env.GetOption('clean'):
    un_pkg_name = open(un_pkg_name_file).read().strip()
else:
    un_pkg_name = ''
un_pkg_files = [[os.path.join(un_home, un_pkg_name),
            'Applications/Folding@home/Uninstall Folding@home.pkg']]

if not env.GetOption('clean'):
    # create dist root for our uninstaller component
    import shutil
    if os.path.exists(un_root): shutil.rmtree(un_root)
    env.CopyToPackage(un_pkg_files, un_root)

    if (not (fah_control_home
        and os.path.exists(fah_control_home + '/build/pkg'))
        and (fah_control_prebuilt_home
        and os.path.exists(fah_control_prebuilt_home + '/build/pkg'))
        ):
        eprint('WARNING: using FAH_CONTROL_PREBUILT_HOME')
        fah_control_home = fah_control_prebuilt_home

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
pkg_components = [
    {'name': 'FAHClient',
        'home': os.environ.get('FAH_CLIENT_HOME'),
        'pkg_id': 'org.foldingathome.fahclient.pkg',
        # must-close seems to require 10.7+
        # close anything that possibly launched or connected to fahclient
        # list of CFBundleIdentifier
        'must_close_apps': [
            'org.foldingathome.fahviewer',
            'org.foldingathome.fahcontrol',
            'edu.stanford.folding.fahviewer',
            'edu.stanford.folding.fahcontrol',
            ],
        # paths relative to 'root'
        'sign_tools': ['usr/local/bin/*'],
        },
    {'name': 'FAHViewer',
        'home': os.environ.get('FAH_VIEWER_HOME'),
        'pkg_id': 'org.foldingathome.fahviewer.pkg',
        'must_close_apps': [
            'org.foldingathome.fahviewer',
            'edu.stanford.folding.fahviewer',
            ],
        'sign_apps': ['Applications/Folding@home/FAHViewer.app'],
        },
    {'name': 'FAHControl',
        'home': fah_control_home,
        'pkg_id': 'org.foldingathome.fahcontrol.pkg',
        'must_close_apps': [
            'org.foldingathome.fahcontrol',
            'edu.stanford.folding.fahcontrol',
            ],
        'sign_apps': ['Applications/Folding@home/FAHControl.app'],
        },
    {'name': 'Uninstaller',
        'home': un_home,
        'pkg_id': 'org.foldingathome.uninstaller.pkg',
        'root': un_root,
        'pkg_files': un_pkg_files,
        },
    ]

name = 'fah-installer'
parameters = {
    'name' : name,
    'version' : version,
    'maintainer' : 'Joseph Coffland <joseph@cauldrondevelopment.com>',
    'vendor' : 'Folding@home',
    'summary' : 'Folding@home',
    'description' : 'Folding@home ' + version + ' Installer Package',
    'pkg_type' : 'dist',
    'pkg_resources' : [['Resources', '.']],
    'pkg_welcome' : 'Welcome.rtf',
    'pkg_license' : 'License.rtf',
    'pkg_background' : 'fah-opacity-50.png',
    'pkg_customize' : 'always',
    'pkg_target' : env.get('osx_min_ver', '10.6'),
    'pkg_components' : pkg_components,
    }
pkg = env.Packager(**parameters)

AlwaysBuild(pkg)
env.Alias('package', pkg)

Clean(pkg, ['build', 'dist', 'config.log', 'package.txt'])
if 'distclean' in COMMAND_LINE_TARGETS:
    Clean('distclean', [
        '.sconsign.dblite', '.sconf_temp', 'config.log',
        'build', 'dist', 'package.txt', Glob(name + '*.pkg'),
        ])
