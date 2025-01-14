---
title: 为Hexo添加Github样式代码高亮
tags:
  - Hexo
  - Github

categories:
  - 教程

cover: https://ik.imagekit.io/xlenco/img/Screenshot_20220903_125917_XPmKbJZQG.jpg?ik-sdk-version=javascript-1.4.3&updatedAt=1662185344028
date: 2022-9-3 14:26
updated: 2022-9-3 15:10
---

![](https://ik.imagekit.io/xlenco/img/Screenshot_20220903_125917_XPmKbJZQG.jpg?ik-sdk-version=javascript-1.4.3&updatedAt=1662185344028)

## 前言

{% note danger modern %}1. 本教程测试环境为 Vercel{% endnote %}
{% note danger modern %}2. 你需要 Github 账号并获取 Personal access tokens，并在 Vercel 中的 Environment Variables 中添加{% endnote %}
{% note danger modern %}获取 Github Personal access tokens 具体教程可查看下方文章{% endnote %}

{% link 点击查看, https://xlenco.eu.org/posts/77e3.html,  https://xlenco.eu.org/img/head.webp %}

## 正文

#### 1. 安装 npm 依赖

```
npm install crypto  --save
npm install xmlhttprequest  --save
```

#### 2. 检查`_config.yml`中`Writing`是否为以下配置

```
highlight:
  enable: false
  line_number: true
  auto_detect: true
  tab_replace: ''
  wrap: true
  hljs: true
prismjs:
  enable: false
  preprocess: true
  line_number: true
  tab_replace: ''
```

这里将 highlight 和 prismjs 关闭即可

#### 3.在` {blogroot}/scripts`处新建 tools.js

````
var XMLHttpRequest = require("xmlhttprequest").XMLHttpRequest;
const crypto = require('crypto');
const { exit } = require('process');
const log = require('hexo-log')({
    debug: false,
    silent: false
  })
// 从环境变量里面拿到github token
const github_token = process.env.GITHUB_TOKEN;

// 找出来所有的描述
var pages = 1;
var XMLHttpRequest = require("xmlhttprequest").XMLHttpRequest;
const crypto = require('crypto');
const { exit } = require('process');
const log = require('hexo-log')({
    debug: false,
    silent: false
  })
// 从环境变量里面拿到github token
const github_token = process.env.GITHUB_TOKEN;

// 找出来所有的描述
var pages = 1;
var is_end = false;
var page_list = [];
var id_list = [];
// for 循环找出所有gist
while (!is_end) {
    // 首先获取到全部的gist
    var request = new XMLHttpRequest();
    log.info(`GIST: 正在查找第${pages}页`)
    request.open("GET", "https://api.github.com/gists?per_page=100" + `&page=${pages}`, false);
    request.setRequestHeader("Content-Type", "application/json");
    request.setRequestHeader("Authorization", "token " + github_token);
    request.send(null);
    data_json = JSON.parse(request.responseText);
    // 如果没有gist就结束
    if (data_json.length == 0) {
        is_end = true;
        break;
    } else {
        // 如果有gist就把gist加入到page_list
        data_json.forEach(element => {
            page_list.push(element["description"]);
            id_list.push(element["id"]);
        });
        pages = pages + 1;
    }
}

// scripts/plantuml.js
const rBacktick = /^((?:[^\S\r\n]*>){0,3}[^\S\r\n]*)(`{3,}|~{3,})[^\S\r\n]*((?:.*?[^`\s])?)[^\S\r\n]*\n((?:[\s\S]*?\n)?)(?:(?:[^\S\r\n]*>){0,3}[^\S\r\n]*)\2[^\S\r\n]?(\n+|$)/gm;
// 另一种正则：```(\w+?\n)([\s\S]+?)```

hexo.extend.filter.register('before_post_render', function (data) {
    const dataContent = data.content;
    if ((!dataContent.includes('```') && !dataContent.includes('~~~'))) return;

    data.content = dataContent.replace(rBacktick, (match, start, $2, _args, _content, end) => {
        let gh_content = _content
            .replace(/\n$/, '')
            .replace(/^>\s/mg, '');
        // 如果有 _args 就用 _args 如果没有就默认为js
        let lang = _args ? _args : 'js';
        lang_list = {
            "javascript": "js",
            "python": "py",
            "SQL": "sql",
            "Python": "py",
            "JavaScript": "js",
            "Javascript": "js",
            "html": "html",
            "HTML": "html",
        }
        lang = lang_list[lang] ? lang_list[lang] : lang;
        const hash = crypto.createHash('md5');
        hash.update(gh_content, 'utf8');
        const md5 = hash.digest('hex');
        process.stdout.write("INFO  |--MD5--|" + md5 + "|----|");
        if (page_list.includes(md5)) {
            // 找到是第几个元素
            let index = page_list.indexOf(md5);
            // 找到对应的id
            var id = id_list[index];
        }else{
            // 如果没有找到就新建一个gist
            console.log("该gist不存在，新建一个gist");
            console.log(gh_content);
            console.log('-------------')
            var id = createGist(md5, gh_content, lang);
            console.log("新建gist，id：" + id);
        }
        console.log('\x1B[32m', "|--ID--|" + id + "|----|")
        // // 查看这个gist的嵌入代码
        // var request = new XMLHttpRequest();
        // let gist_url = "https://gist.github.com/zkeq/" + id + ".js"
        // console.log(gist_url);
        // request.open("GET", gist_url, false);
        // request.send();
        // let gist_code = request.responseText;
        // str = "<script>" + gist_code + "</script>";
            // iframe_ele = document.createElement('iframe');
            // iframe_ele.style = "border:none;width:100%;max-height:50vh";
            // iframe_ele.setAttribute("onload", "javascript:this.style.height=`${this.contentWindow.document.body.offsetHeight}px`;this.contentWindow.document.getElementsByClassName('gist-data')[0].style.height=`${this.clientHeight-65}px`;")
            // iframe_ele.setAttribute("srcdoc", `<head><base target='_blank'/></head><body><script src='https://gist.github.com/zkeq/${id}.js'></script></body>`)
        return `

<iframe
style="border:none;width:100%;;max-height:66vh;"
onload="javascript:this.style.height=\`\${this.contentWindow.document.body.offsetHeight+20}px\`;this.contentWindow.document.getElementsByClassName('gist-data')[0].style.height=\`\${this.clientHeight-55}px\`;"
srcdoc='<meta name="description" content="Instantly share code, notes, and snippets. You can&#39;t perform that action at this time. You signed in with another tab or window. You signed out in another tab or window. Reload to refresh your session. Reload to refresh your session."><meta http-equiv="Content-Type" content="text/html; charset=utf-8"><meta name="twitter:widgets:csp" content="on"><meta name="robots" content="noindex"><base target="_blank"><style>body {text-rendering: optimizeLegibility; -webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale; font-family: "ff-tisa-web-pro", Georgia, Cambria, "Times New Roman", Times, serif; font-weight: 400; color: #333332; font-size: 18px; line-height: 1.4; margin: 0; background-color: white; overflow: hidden;}iframe {max-width: 100%;}</style></head><body><style>.gist .gist-file { margin-bottom: 0 !important; }.gist { text-rendering: auto; }.gist-meta a:nth-child(2) {display: none;} html {overflow: hidden;text-overflow: ellipsis;display: inline-block;} * {margin: 0}</style><script src="https://gist.onmicrosoft.cn/zkeq/${id}.js" charset="utf-8"></script><script>var height = -1; var delayMs = 200; if (document) {document.domain = document.domain;}function notifyResize(height) {height = height ? height : document.documentElement.offsetHeight; var resized = false; if (window.donkey && donkey.resize) {donkey.resize(height);var elements = document.getElementsByClassName("gist-data"); for (var i = 0; i < elements.length; i++) {elements[i].style.overflow = "visible"}resized = true;}if (parent && parent._resizeIframe) {var obj = {iframe: window.frameElement, height: height}; parent._resizeIframe(obj); resized = true;}if (window.location && window.location.hash === "#amp=1" && window.parent && window.parent.postMessage) {window.parent.postMessage({sentinel: "amp", type: "embed-size", height: height}, "*");}if (window.webkit && window.webkit.messageHandlers && window.webkit.messageHandlers.resize) {window.webkit.messageHandlers.resize.postMessage(height); resized = true;}return resized;}function maybeResize() {try {if (document.documentElement.offsetHeight != height && notifyResize()) {height = document.documentElement.offsetHeight;}delayMs = Math.min(delayMs * 2, 1000000); setTimeout(maybeResize, delayMs);} catch(error) {console.log("maybeResize error: ", error)}}maybeResize();</script>'
>
</iframe>

`;
    });
}, 9);

function createGist (md5, gh_content, lang){
    let response = ''
    let requestData = {
        "description": md5,
        "public": false,
        "files": {
            [md5 + "." + lang]: {
                "content": gh_content
            }
        }
    };
    var request = new XMLHttpRequest();
    // POST
    request.open("POST", "https://api.github.com/gists", false);
    request.setRequestHeader("Content-Type", "application/json");
    request.setRequestHeader("Authorization", "token " + github_token);
    request.send(JSON.stringify(requestData));
    response = request.responseText;
    // 如果状态码不是200就报错
    if (request.status != 201) {
        return createGist(md5, gh_content, lang);
    }
    return JSON.parse(response)["id"];
}

````

{% note warning modern %}将上文内容中的 zkeq 改为你的 Github 用户名即可{% endnote %}

#### 4. 在 Vercel 的 Environment Variables 中添加变量

| VALUE        | NAME                         |
| ------------ | ---------------------------- |
| GITHUB_TOKEN | 这里写你 GITHUB_TOKEN 的内容 |

{% note info modern %}第一次部署可能速度较慢，请耐心等待{% endnote %}
