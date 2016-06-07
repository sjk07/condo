---
layout: docs
title: node-download
group: shades
---

@{/*

node-download
    Downloads and installs node if it is not already installed either locally or globally.

node_download_path='$(base_path)/bin/nodejs'
    The path in which to install node.

base_path='$(CurrentDirectory)'
    The base path in which to install node.

*/}

use assembly = 'System.IO.Compression.FileSystem, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'

use namespace = 'System'
use namespace = 'System.IO'
use namespace = 'System.IO.Compression'
use namespace = 'System.Net'

use import = 'Condo.Build'

default base_path               = '${ Directory.GetCurrentDirectory() }'

default node_download_path      = '${ Path.Combine(base_path, "bin", "nodejs") }'
default node_version            = '0.10.28'
default npm_version             = '1.4.9'

default node_hash               = '628FFD6C3577068C00CEC9F6F897F0EC8F5212D9'

@{
    Build.Log.Header("node-download");

    var node_exe                = "node.exe";
    var node_dist               = "http://nodejs.org/dist";
    var node_url                = node_dist + "/v" + node_version + "/" + node_exe;
    var node_install            = Path.Combine(node_download_path, node_exe);

    var npm_zip                 = "npm-" + npm_version + ".zip";
    var npm_url                 = node_dist + "/npm/" + npm_zip;
    var npm_install             = Path.Combine(node_download_path, npm_zip);
    var npm_cmd_install         = Path.Combine(node_download_path, "npm.cmd");

    var node_locally_installed  = File.Exists(node_install);
    var node_globally_installed = Build.TryExecute("node", out node_version, "--version");
    var npm_globally_installed  = Build.TryExecute("npm", out npm_version, "--version");
    var node_installed          = node_globally_installed || node_locally_installed;

    node_install = node_globally_installed || Build.Unix ? "node" : node_install;
    npm_cmd_install = npm_globally_installed || Build.Unix ? "npm" : npm_cmd_install;

    if (!node_installed)
    {
        if (Build.OSX)
        {
            node_install = "node";
            npm_cmd_install = "npm";
        }
        else if (Build.Unix)
        {
            throw new Exception("node-download: you must install node manually for Linux environments.");
        }
        else
        {
            Log.Info(string.Format("--installing nodejs to: {0}", node_download_path));

            if (Directory.Exists(node_download_path))
            {
                Directory.Delete(node_download_path, recursive: true);
            }

            Directory.CreateDirectory(node_download_path);

            using(var client = new WebClient())
            {
                Log.Info(string.Format("--downloading nodejs from: {0} to: {1}", node_url, node_install));
                client.DownloadFile(node_url, node_install);

                Log.Info(string.Format("--downloading npm from: {0} to: {1}", npm_url, npm_install));
                client.DownloadFile(npm_url, npm_install);

                Log.Info(string.Format("--extracting npm from: {0} to: {1}", npm_install, node_download_path));
                ZipFile.ExtractToDirectory(npm_install, node_download_path);
            }
        }
    }

    // attempt to retreive node version
    Build.TryExecute(node_install, out node_version, "--version");

    // attempt to retrieve npm version
    Build.TryExecute(npm_cmd_install, out npm_version, "--version");

    Build.SetPath("node", node_install, node_globally_installed);
    Build.SetPath("npm", npm_cmd_install, node_globally_installed);
}

verify-hash verify_assembly='${ node_install }' verify_hash='${ node_hash }' if='!node_installed && !Build.Unix'
brew-install brew_install_formula='node' if='!node_installed && Build.OSX'

@{
    Build.Log.Argument("node path", node_install);
    Build.Log.Argument("npm path", npm_cmd_install);
    Build.Log.Argument("version", node_version);
    Build.Log.Argument("npm version", npm_version);
    Build.Log.Header();
}