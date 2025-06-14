<div align="center">
    
<img src="docs/images/assimpjs_logo_small.png?raw=true"> <br>

[Check out the live demo here!](http://kovacsv.github.io/assimpjs)

[![npm version](https://badge.fury.io/js/assimpjs.svg)](https://badge.fury.io/js/assimpjs)
[![Native Build](https://github.com/kovacsv/assimpjs/actions/workflows/native_build.yml/badge.svg)](https://github.com/kovacsv/assimpjs/actions/workflows/native_build.yml)
[![WASM Build](https://github.com/kovacsv/assimpjs/actions/workflows/wasm_build.yml/badge.svg)](https://github.com/kovacsv/assimpjs/actions/workflows/wasm_build.yml)
    
</div>

# assimpjs

The [emscripten](https://emscripten.org) interface for the [assimp](https://github.com/assimp/assimp) library. It runs entirely in the browser, and allows you to import 40+ 3D file formats and access the result in JSON or glTF format. This is not a full port of assimp, but an easy to use interface to access it's functionality.

## How to install?

You can get assimpjs from [npm](https://www.npmjs.com/package/assimpjs):

```
npm install assimpjs
```

## How to use?

The library runs in the browser and as a node.js module as well.

You will need two files from the `dist` folder: `assimpjs.js` and `assimpjs.wasm`. The wasm file is loaded runtime by the js file.

Given that browsers don't access the file system, you should provide all the files needed for import. Some 3D formats are coming in multiple files, so you should list all of them to import the model properly.

You should provide two things for every file:
- **name:** The name of the file. It's used if files are referring to each other.
- **content:** The content of the file as an `Uint8Array` object.

The supported target formats are: `assjson`, `gltf`, `gltf2`, `glb`, and `glb2`. The number of result files depends on the format.

### Use from the browser

First, include the `assimpjs.js` file in your website.

```html
<script type="text/javascript" src="assimpjs.js"></script>
```

After that, download the model files, and pass them to assimpjs.

```js
assimpjs ().then (function (ajs) {
    // fetch the files to import
    let files = [
        'testfiles/cube_with_materials.obj',
        'testfiles/cube_with_materials.mtl'
    ];
    Promise.all (files.map ((file) => fetch (file))).then ((responses) => {
        return Promise.all (responses.map ((res) => res.arrayBuffer ()));
    }).then ((arrayBuffers) => {
        // create new file list object, and add the files
        let fileList = new ajs.FileList ();
        for (let i = 0; i < files.length; i++) {
            fileList.AddFile (files[i], new Uint8Array (arrayBuffers[i]));
        }
        
        // convert file list to assimp json
        let result = ajs.ConvertFileList (fileList, 'assjson');
        
        // check if the conversion succeeded
        if (!result.IsSuccess () || result.FileCount () == 0) {
            resultDiv.innerHTML = result.GetErrorCode ();
            return;
        }

        // get the result file, and convert to string
        let resultFile = result.GetFile (0);
        let jsonContent = new TextDecoder ().decode (resultFile.GetContent ());

        // parse the result json
        let resultJson = JSON.parse (jsonContent);
        
        resultDiv.innerHTML = JSON.stringify (resultJson, null, 4);
    });
});
```

### Use as a node.js module

You should require the `assimpjs` module in your script. In node.js you can use the file system module to get the buffer of each file.

```js
let fs = require ('fs');
const assimpjs = require ('assimpjs')();

assimpjs.then ((ajs) => {
    // create new file list object
    let fileList = new ajs.FileList ();
    
    // add model files
    fileList.AddFile (
        'cube_with_materials.obj',
        fs.readFileSync ('testfiles/cube_with_materials.obj')
    );
    fileList.AddFile (
        'cube_with_materials.mtl',
        fs.readFileSync ('testfiles/cube_with_materials.mtl')
    );
    
    // convert file list to assimp json
    let result = ajs.ConvertFileList (fileList, 'assjson');

    // check if the conversion succeeded
    if (!result.IsSuccess () || result.FileCount () == 0) {
        console.log (result.GetErrorCode ());
        return;
    }

    // get the result file, and convert to string
    let resultFile = result.GetFile (0);
    let jsonContent = new TextDecoder ().decode (resultFile.GetContent ());

    // parse the result json
    let resultJson = JSON.parse (jsonContent);
});
```

It's also possible to delay load the required files so they have to be loaded only when the importer needs them. In this case you have to provide only the name and content of the main file, and implement callbacks to provide additional files.

```js
let fs = require ('fs');
const assimpjs = require ('assimpjs')();

assimpjs.then ((ajs) => {
    // convert model
    let result = ajs.ConvertFile (
        // file name
        'cube_with_materials.obj',
        // file format
        'assjson',
        // file content as arraybuffer
        fs.readFileSync ('testfiles/cube_with_materials.obj'),
        // check if file exists by name
        function (fileName) {
            return fs.existsSync ('testfiles/' + fileName);
        },
        // get file content as arraybuffer by name
        function (fileName) {
            return fs.readFileSync ('testfiles/' + fileName);
        }
    );
    
    // check if the conversion succeeded
    if (!result.IsSuccess () || result.FileCount () == 0) {
        console.log (result.GetErrorCode ());
        return;
    }

    // get the result file, and convert to string
    let resultFile = result.GetFile (0);
    let jsonContent = new TextDecoder ().decode (resultFile.GetContent ());

    // parse the result json
    let resultJson = JSON.parse (jsonContent);
});
```

## How to build on Windows?

A set of batch scripts are prepared for building on Windows.

### 1. Install Prerequisites

Install [CMake](https://cmake.org) (3.6 minimum version is needed). Make sure that the cmake executable is in the PATH.

### 2. Install Emscripten SDK

Run the Emscripten setup script.

```
tools\setup_emscripten_win.bat
```

### 3. Compile the WASM library

Run the release build script.

```
tools\build_wasm_win_release.bat
```

### 4. Build the native project (optional)

If you want to debug the code, it's useful to build a native project. To do that, just use cmake to generate the project of your choice.

## How to run locally?

To run the demo and the examples locally, you have to start a web server. Run `npm install` from the root directory, then run `npm start` and visit `http://localhost:8080`.

# Changes in this fork

To use this, simply import the library in html and set wasm path - 

```html
<script src="https://cdn.jsdelivr.net/gh/repalash/assimpjs@fbx/dist/assimpjs.js"></script>
```
```typescript
const ajs = await assimpjs({
    locateFile: (file: string) => 'https://cdn.jsdelivr.net/gh/repalash/assimpjs@fbx/dist/' + file,
})
```

Changes -
- [x] Build with FBX export support
- [ ] Build without import formats that are not supported properly in three.js. (and gltf)
- [ ] Build with all export formats not in three.js (and gltf)
- [ ] Run in web worker

## Three.js Export to FBX

This can be used to export and download fbx files from three.js scenes. This is done by first exporting to glTF, then converting to FBX using assimpjs.

```js
// Export a three.js scene to FBX using assimpjs (vanilla JS)
import { GLTFExporter } from 'three/examples/jsm/exporters/GLTFExporter.js';

const exporter = new GLTFExporter();
exporter.parse(scene, function (gltf) {
    // Convert glTF JSON to Uint8Array
    const gltfStr = JSON.stringify(gltf);
    const gltfBytes = new TextEncoder().encode(gltfStr);

    let fileList = new ajs.FileList();
    fileList.AddFile('scene.gltf', gltfBytes);
    let result = ajs.ConvertFileList(fileList, 'fbx');
    if (result.IsSuccess() && result.FileCount() > 0) {
        let fbxFile = result.GetFile(0);
        let blob = new Blob([fbxFile.GetContent()], {type: 'application/octet-stream'});
        let url = URL.createObjectURL(blob);
        let a = document.createElement('a');
        a.href = url;
        a.download = 'scene.fbx';
        a.click();
        URL.revokeObjectURL(url);
    }
});
```

Note - set the export option `embedImages: true`(depending on three.js version) in the `GLTFExporter` to embed images in the glTF file, otherwise you will need to provide the images separately.

It can also be used in three.js to load unsupported file formats, convert to gltf2 and then use the `GLTFLoader` to load the model into a three.js scene.

```js
// Load an unsupported file (e.g. .obj) into three.js via assimpjs and GLTFLoader
fetch('model.obj')
    .then(res => res.arrayBuffer())
    .then(objBuffer => {
    let fileList = new ajs.FileList();
    fileList.AddFile('model.obj', new Uint8Array(objBuffer));
    let result = ajs.ConvertFileList(fileList, 'gltf2');
    if (result.IsSuccess() && result.FileCount() > 0) {
        let gltfFile = result.GetFile(0);
        let blob = new Blob([gltfFile.GetContent()], {type: 'application/json'});
        let url = URL.createObjectURL(blob);
        const loader = new THREE.GLTFLoader();
        loader.load(url, gltf => {
            scene.add(gltf.scene);
            URL.revokeObjectURL(url);
        });
    }
});
```

## Threepipe

This is available as a [package](https://threepipe.org/package/plugin-assimpjs.html) in [threepipe](https://threepipe.org) and [webgi](https://webgi.dev) and can be used directly to load many file formats and export to fbx, assjson etc.
