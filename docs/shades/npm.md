---
layout: docs
title: npm
group: shades
---

@{/*

npm
    Executes a node package manager (npm) command.

npm_args=''
    Required. The arguments to pass to the npm command.

npm_options='' (Environment Variable: NPM_OPTIONS)
    Additional options to use when executing the npm command.

npm_path='$(working_path)'
    The path in which to execute the npm command.

working_path='$(base_path)'
    The working path in which to execute the npm command.

base_path='$(CurrentDirectory)'
    The base path in which to execute the npm command.

npm_wait='true'
    A value indicating whether or not to wait for exit.

npm_quiet='$(Build.Log.Quiet)'
    A value indicating whether or not to avoid printing output.

npm_loglevel='error'
    The log level to use when calling npm: (silent, error, warn, http, info, verbose, silly)

*/}

use namespace = 'System'

use import = 'Condo.Build'

default base_path       = '${ Directory.GetCurrentDirectory() }'
default working_path    = '${ base_path }'

default npm_args        = ''
default npm_options     = '${ Build.Get("NPM_OPTIONS") }'
default npm_path        = '${ working_path }'
default npm_wait        = '${ true }'
default npm_quiet       = '${ Build.Log.Quiet }'
default npm_loglevel    = 'error'

node-download once='node-download'

@{
    Build.Log.Header("npm");

    var npm_cmd = Build.GetPath("npm");

    if (string.IsNullOrEmpty(npm_args))
    {
        throw new ArgumentException("npm: cannot be executed without arguments.", "npm_args");
    }

    Build.Log.Argument("cli", npm_cmd.Path);
    Build.Log.Argument("arguments", npm_args);
    Build.Log.Argument("options", npm_options);
    Build.Log.Argument("path", npm_path);
    Build.Log.Argument("wait", npm_wait);
    Build.Log.Argument("quiet", npm_quiet);
    Build.Log.Argument("log level", npm_loglevel);
    Build.Log.Header();

    if (!string.IsNullOrEmpty(npm_loglevel))
    {
        npm_options += " --loglevel " + npm_loglevel;
    }

    npm_options = npm_options.Trim();
}

exec exec_cmd='${ npm_cmd.Path }' exec_args='${ npm_options } ${ npm_args }' exec_path='${ npm_path }' exec_wait='${ npm_wait }' exec_quiet='${ npm_quiet }' exec_redirect='${ false }'