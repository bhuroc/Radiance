
import os
import sys
import string

PATHFILE = 'scbuild/raypaths.py'
OPTFILE = 'scbuild/rayopts.py'
def set_pre_opts():
	vars = Variables(OPTFILE, ARGUMENTS)
	vars.Add('SKIP', 'Skip Display of License terms', 0)
	vars.Add('RAD_DEBUG',  'Build a debug version',  0)
	vars.Add('MSVC_VERSION', 'Microsoft VC Version',  '12.0')
	vars.Add('TARGET_ARCH',  'Windows only: "x68" or "amd64"',  '')
	return vars

def set_opts(env):
	vars = Variables(PATHFILE, ARGUMENTS)
	vars.Add('RAD_BINDIR',  'Install executables here',   env['RAD_BINDIR'])
	vars.Add('RAD_RLIBDIR', 'Install support files here', env['RAD_RLIBDIR'])
	vars.Add('RAD_MANDIR',  'Install man pages here',     env['RAD_MANDIR'])
	vars.Update(env) 
	vars.Save(PATHFILE, env)
	Help(vars.GenerateHelpText(env, sort=cmp))
	# where stuff is located in the source tree
	# the binary target libs are configured by platform
	env['RAD_BUILDRLIB'] = '#lib'
	env['RAD_BUILDMAN']  = '#doc/man'

def allplats_setup(env):
	from build_utils import find_libs
	find_libs.find_radlib(env)
	find_libs.find_x11(env)
	find_libs.find_gl(env) # OpenGL
	find_libs.find_libtiff(env)
	find_libs.find_pyinstaller(env)

def post_common_setup(env):
	env.Append(CPPPATH = [os.path.join('#src', 'common')])
	env.Append(LIBPATH=[env['RAD_BUILDLIB']]) # our own libs
	if not env.has_key('RAD_MLIB'):
		env['RAD_MLIB'] = [] #['m'] # no seperate mlib on Win

def shareinstall_setup(env):
	from build_utils import install
	# only scan for those files when we actually need them
	if 'install' in sys.argv or 'rlibinstall' in sys.argv:
		install.install_rlibfiles(env)
	if 'install' in sys.argv or 'maninstall' in sys.argv:
		install.install_manfiles(env)


# set stuff before loading platforms
prevars = set_pre_opts()
# Set up build environment
env = Environment(variables=prevars)
prevars.Save(OPTFILE, env)
env.Decider('timestamp-match')

from build_utils import install
script_b = Builder(action = install.install_script, suffix = '')
env.Append(BUILDERS={'InstallScript': script_b})
if os.name == 'posix':
	tclscript_b = Builder(action = install.install_tclscript, suffix = '')
	env.Append(BUILDERS={'InstallTCLScript': tclscript_b})


# configure platform-specific stuff
from build_utils import load_plat
load_plat.load_plat(env)

# override options
set_opts(env)

# accept license
if ((not env['SKIP']
			or env['SKIP'].strip().lower() in (0,'0','n','no','false',None))
		and not '-c' in sys.argv and not 'test' in sys.argv):
	from build_utils import copyright
	copyright.show_license()

# fill in generic config
allplats_setup(env)

# Bring in all the actual things to build
Export('env')
if 'test' in sys.argv:
	SConscript(os.path.join('test', 'SConscript'))
	
else:
	SConscript(os.path.join('src', 'common', 'SConscript'),
			variant_dir=os.path.join(env['RAD_BUILDOBJ'],'common'), duplicate=0)
	post_common_setup(env)
	dirs = Split('''meta cv gen ot rt px hd util cal''')
	if os.path.isfile('src/winimage/SConscript'):
		dirs.append('winimage')
	if os.path.isfile('src/winrview/SConscript'):
		dirs.append('winrview')
	for d in dirs:
		print(d)
		SConscript(os.path.join('src', d, 'SConscript'),
				variant_dir=os.path.join(env['RAD_BUILDOBJ'], d), duplicate=0)

	if string.find(string.join(sys.argv[1:]), 'install') > -1:
		shareinstall_setup(env)

Default('.')

# virtual targets
env.Alias('bininstall',  '$RAD_BINDIR')
env.Alias('rlibinstall', '$RAD_RLIBDIR')
env.Alias('maninstall',  '$RAD_MANDIR')

env.Alias('build',   ['$RAD_BUILDBIN', '$RAD_BUILDRLIB'])
env.Alias('test',    ['#test'])
env.Alias('install', ['bininstall', 'rlibinstall', 'maninstall'])

# vim: set syntax=python:
