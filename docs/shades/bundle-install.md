---
layout: docs
title: bundle-install
group: shades
---

@{/*

bundle-install
    Installs a package using the bundler command line utility for ruby.

bundle_install_frozen='true'
    A value indicating whether or not to update the Gemfile.lock file after install.

bundle_install_options='' (Environment Variable: BUNDLE_INSTALL_OPTIONS)
    Additional options to use when executing the bundle install command.

bundle_install_path='$(working_path)'
    The path in which to execute the bundle install command.

base_path='$(CurrentDirectory)'
    The base path in which to execute the bundle install command.

working_path='$(base_path)'
    The working path in which to execute the bundle install command.

*/}

use namespace = 'System'
use namespace = 'System.IO'

use import = 'Condo.Build'

default base_path                   = '${ Directory.GetCurrentDirectory() }'
default working_path                = '${ base_path }'

default bundle_install_frozen       = '${ false }' type='bool'
default bundle_install_options      = '${ Build.Get("BUNDLE_INSTALL_OPTIONS") }'
default bundle_install_path         = '${ working_path }'

@{
    Build.Log.Header("bundle-install");

    // trim the arguments
    bundle_install_options = bundle_install_options.Trim();

    Build.Log.Argument("frozen", bundle_install_frozen);
    Build.Log.Argument("options", bundle_install_options);
    Build.Log.Argument("path", bundle_install_path);
    Build.Log.Header();

    // determine if the gemfile.lock should be frozen
    if (bundle_install_frozen)
    {
        // add the frozen option
        bundle_install_options = (bundle_install_options + " --frozen").Trim();
    }
}

bundle bundle_args='install' bundle_options='${ bundle_install_options }' bundle_path='${ bundle_install_path }'