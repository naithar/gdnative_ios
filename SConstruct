#!/usr/bin/env python
import os
import sys
import subprocess

if sys.version_info < (3,):
    def decode_utf8(x):
        return x
else:
    import codecs
    def decode_utf8(x):
        return codecs.utf_8_decode(x)[0]

# Most of the settings are taken from https://github.com/BastiaanOlij/gdnative_cpp_example

opts = Variables([], ARGUMENTS)

# Gets the standard flags CC, CCX, etc.
env = DefaultEnvironment()

# Define our options
opts.Add(EnumVariable('target', "Compilation target", 'debug', ['d', 'debug', 'r', 'release']))
opts.Add(EnumVariable('platform', "Compilation platform", '', ['', 'ios']))
opts.Add(EnumVariable('p', "Compilation target, alias for 'platform'", '', ['','ios']))
opts.Add(EnumVariable('arch', "Compilation platform", '', ['', 'arm64', 'armv7', 'x86_64']))
opts.Add(BoolVariable('use_llvm', "Use the LLVM / Clang compiler", 'no'))
opts.Add(PathVariable('target_path', 'The path where the lib is installed.', 'bin/'))
opts.Add(PathVariable('target_name', 'The library name.', 'gdexample', PathVariable.PathAccept))
opts.Add(EnumVariable('mode', 'Library build mode', 'static', ['static', 'dynamic']))

# Local dependency paths, adapt them to your setup
godot_headers_path = "godot-cpp/godot_headers/"
cpp_bindings_path = "godot-cpp/"
cpp_library = "gdnative.fat.a"

# Updates the environment with the option variables.
opts.Update(env)

# Process some arguments
if env['use_llvm']:
    env['CC'] = 'clang'
    env['CXX'] = 'clang++'

if env['p'] != '':
    env['platform'] = env['p']

if env['platform'] == '':
    print("No valid target platform selected.")
    quit();

# For the reference:
# - CCFLAGS are compilation flags shared between C and C++
# - CFLAGS are for C-specific compilation flags
# - CXXFLAGS are for C++-specific compilation flags
# - CPPFLAGS are for pre-processor flags
# - CPPDEFINES are for pre-processor defines
# - LINKFLAGS are for linking flags

# Check our platform specifics
if env['platform'] == "ios":
    env.Append(CCFLAGS=["-fmodules", "-fcxx-modules"])

    if env['arch'] == 'x86_64':
        sdk_name = 'iphonesimulator'
        env.Append(CCFLAGS=['-mios-simulator-version-min=10.0'])
    else:
        sdk_name = 'iphoneos'
        env.Append(CCFLAGS=['-miphoneos-version-min=10.0'])

    try:
        sdk_path = decode_utf8(subprocess.check_output(['xcrun', '--sdk', sdk_name, '--show-sdk-path']).strip())
    except (subprocess.CalledProcessError, OSError):
        raise ValueError("Failed to find SDK path while running xcrun --sdk {} --show-sdk-path.".format(sdk_name))
    
    env['target_path'] += 'ios/'
    env.Append(CCFLAGS=['-arch', env['arch'], "-isysroot", "$IPHONESDK", "-stdlib=libc++", '-isysroot', sdk_path])
    env.Append(CXXFLAGS=['-std=c++17'])

    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS=['-g', '-O2'])
    else:
        env.Append(CCFLAGS=['-g', '-O3'])
    

    env.Append(
        CCFLAGS="-fmessage-length=0 -fno-strict-aliasing -fdiagnostics-print-source-range-info -fdiagnostics-show-category=id -fdiagnostics-parseable-fixits -fpascal-strings -fblocks -fvisibility=hidden -MMD -MT dependencies -miphoneos-version-min=10.0".split()
    )
    env.Append(LINKFLAGS=[])

    env.Append(
        LINKFLAGS=[
            "-arch", 
            env['arch'],
            "-miphoneos-version-min=10.0",
            '-isysroot', sdk_path,
            '-F' + sdk_path
        ]
    )

# make sure our binding library is properly includes
env.Append(CPPPATH=['.', godot_headers_path, cpp_bindings_path + 'include/', cpp_bindings_path + 'include/core/', cpp_bindings_path + 'include/gen/'])
env.Append(LIBPATH=[cpp_bindings_path + 'bin/'])
env.Append(LIBS=[cpp_library])

# tweak this if you want to use different folders, or more folders, to store your source code in.
env.Append(CPPPATH=['src/'])
sources = Glob('gdnative_ios/src/*.cpp')
sources.append(Glob("gdnative_ios/src/*.mm"))
sources.append(Glob("gdnative_ios/src/*.m"))

if env['mode'] == 'static':
    library = env.StaticLibrary(target=env['target_path'] + env['target_name'] , source=sources)
else:
    library = env.SharedLibrary(target=env['target_path'] + env['target_name'] , source=sources)

Default(library)

# Generates help for the -h scons option.
Help(opts.GenerateHelpText(env))
