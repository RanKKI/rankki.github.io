---
title: "Tree Shaking Cocos Creator 项目"
date: 2021-09-16T10:32:58+08:00
draft: false
categories:
 - 技术
tags:
 - Cocos Creator
---

Cocos Creator 打包出的项目，会对每个文件生成 `helper function`，但实际上，我们只有一个 `index.js`，在打包的时候却把所有的 `helper function` 打到了一起，因此导致一个 `index.js` 文件内，拥有很多个 `helper function`

因此可以通过 Tree Shaking 的方法能裁切掉很多无用的函数定义，从而降低文件大小. 

<!--more-->

> 通过改方法，可以将一个原本 200KB 的代码文件，可以被缩减到 110KB

```javascript
'use strict';

const path = require('path');
const ts = require("rollup-plugin-typescript2");
const glob = require("glob")
const fs = require("fs-extra")
const commonjs = require('@rollup/plugin-commonjs');

const rollup = require('rollup');
const { terser } = require('rollup-plugin-terser')

function getFileSize(path) {
    const stats = fs.statSync(path)
    const fileSizeInBytes = stats.size;
    const ret = fileSizeInBytes / 1024
    return ret.toFixed(4)
}

async function treeShakeCocos(options) {
    Editor.log("shake index.js start")
    const destPath = options.dest
    const indexJS = path.join(destPath, "assets", "main", "index.js")
    const oldSize = getFileSize(indexJS)
    try {
        const bundle = await rollup.rollup({
            input: "entrypoint.js",
            plugins: [
                commonjs(),
                createEntryPointPlugin(options),
                ts({
                    cwd: path.join(".", "packages", "wechatgame_build"),
                    tsconfig: "tsconfig.rollup.json",
                    check: false,
                }),
                cocosUUIDPlugin(),
            ],
        })
        await bundle.write({
            file: indexJS,
            format: "es",
            minify: true,
            plugins: [
                terser({
                    output: {
                        comments: false,
                        ascii_only: true, // 仅输出ascii字符
                        beautify: false,// 最紧凑的输出
                    },
                    compress: {
                        collapse_vars: true, // 内嵌定义了但是只用到一次的变量
                        reduce_vars: true, // 提取出出现多次但是没有定义成变量去引用的静态值
                    },
                }),
            ],
        })
        await bundle.close()
        const newSize = getFileSize(indexJS)
        Editor.log("shake index.js end, reduce size from", oldSize + " KB", "to", newSize + " KB")
    } catch (err) {
        Editor.log("Failed to tree shake cocos project", err)
    }
}

async function onBuildFinished(options, callback) {
    process.chdir(options.project)
    await treeShakeCocos(options)
    callback()
}

function createEntryPointPlugin(options) {
    return {
        name: "create-entry-point",
        resolveId(source) {
            if (source === "entrypoint.js") {
                return source;
            }
            return null;
        },
        load(id) {
            if (id === "entrypoint.js") {
                return createEntryPoint(options);
            }
            return null;
        },
    };
}

async function createEntryPoint(options) {
    const files = await Promise.all([
        fetchCode(options, "scripts"),
    ]);
    const imports = files.flat().map((file) => `import "./${file}"`);
    return imports.flat().join("\n");
}

async function fetchCode(options, base) {
    return new Promise((resolve, reject) => {
        glob(`assets/${base}/**/!(*.d).ts`, function (err, files) {
            if (err) {
                reject(err);
            } else {
                resolve(files);
            }
        });
    });
}

function cocosUUIDPlugin() {
    return {
        name: "create-entry-point",
        resolveId(source) {
            return null;
        },
        async transform(code, id) {
            if (id.endsWith(".ts") && !id.endsWith("cchelper.ts") && /cc\._decorator/.test(code)) {
                const uuid = await loadUUID(id)
                const result = [
                    `cc._RF.push({exports:{}}, "${uuid}", "${/.*\/([^\/]*)\.ts/.exec(id)[1]}");`,
                    code,
                    `cc._RF.pop();`
                ].join("\n")
                return result;
            }
            return null;
        },
    };
}

async function loadUUID(codePath) {
    const metaSrc = await fs.readFile(codePath + ".meta", "utf-8");
    const mate = JSON.parse(metaSrc);
    return c.compressUuid(mate.uuid, false);
}

let t = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
let r = new Array(128);
for (let i = 0; i < 128; ++i) {
    r[i] = 0;
}
for (i = 0; i < 64; ++i){
    r[t.charCodeAt(i)] = i;
}

const n = /-/g,
    s = /^[0-9a-fA-F-]{36}$/,
    o = /^[0-9a-fA-F]{32}$/;
const c = {
    compressUuid: function (e, t) {
        if (s.test(e)) e = e.replace(n, "");
        else if (!o.test(e)) return e;
        let r = !0 === t ? 2 : 5;
        return c.compressHex(e, r);
    },
    compressHex: function (e, r) {
        let n = e.length;
        let i = void 0 !== r ? r : n % 3;
        for (var s = e.slice(0, i), o = []; i < n;) {
            var u = parseInt(e[i], 16),
                a = parseInt(e[i + 1], 16),
                c = parseInt(e[i + 2], 16);
            o.push(t[(u << 2) | (a >> 2)]), o.push(t[((3 & a) << 4) | c]), (i += 3);
        }
        return s + o.join("");
    },
    decompressUuid: function (e) {
        if (23 === e.length) {
            let t = [];
            for (let i = 5; i < 23; i += 2) {
                let n = r[e.charCodeAt(i)],
                    s = r[e.charCodeAt(i + 1)];
                t.push((n >> 2).toString(16)),
                    t.push((((3 & n) << 2) | (s >> 4)).toString(16)),
                    t.push((15 & s).toString(16));
            }
            e = e.slice(0, 5) + t.join("");
        } else {
            if (22 !== e.length) return e;
            {
                let t = [];
                for (let i = 2; i < 22; i += 2) {
                    let n = r[e.charCodeAt(i)],
                        s = r[e.charCodeAt(i + 1)];
                    t.push((n >> 2).toString(16)),
                        t.push((((3 & n) << 2) | (s >> 4)).toString(16)),
                        t.push((15 & s).toString(16));
                }
                e = e.slice(0, 2) + t.join("");
            }
        }
        return [e.slice(0, 8), e.slice(8, 12), e.slice(12, 16), e.slice(16, 20), e.slice(20)].join("-");
    },
};


module.exports = {
    load() {
        Editor.Builder.on("build-finished", onBuildFinished)
    },

    unload() {
        Editor.Builder.removeListener("build-finished", onBuildFinished)
    },
};



```