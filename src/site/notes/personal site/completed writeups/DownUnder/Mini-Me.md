---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/down-under/mini-me/","tags":["web","downunder_ctf_25"]}
---

The main page has a brainrot video and some js code on it that mentions a js map which is in `/static/js`:

![Pasted image 20250817083910.png](/img/user/img/Pasted%20image%2020250817083910.png)

Map here:
```js
{
    "version": 3,
    "file": "main.min.js.map",
    "sources": ["main.js"],
    "sourcesContent": [
        "function pingMailStatus() {\r\n  fetch(\"/api/mail/status\");\r\n}\r\n\r\nfunction fetchInboxPreview() {\r\n  fetch(\"/api/mail/inbox?limit=5\");\r\n}\r\n\r\npingMailStatus();\r\nfetchInboxPreview();\r\n\r\ndocument.getElementById(\"start-btn\")?.addEventListener(\"click\", () => {\r\n  const audio = document.getElementById(\"balletAudio\");\r\n  audio.play();\r\n\r\n  document.getElementById(\"start-btn\").style.display = \"none\";\r\n  document.getElementById(\"audio-warning\").style.display = \"none\";\r\n\r\n  const dancer = document.getElementById(\"dancer\");\r\n  const dancerImg = document.getElementById(\"dancer-img\"); // Get the image element\r\n\r\n  dancer.style.display = \"block\";\r\n  dancerImg.style.display = \"block\"; // Show the image\r\n\r\n  let angle = 0;\r\n  const radius = 100;\r\n  const centerX = window.innerWidth / 2;\r\n  const centerY = window.innerHeight / 2;\r\n\r\n  function animate() {\r\n    angle += 0.05;\r\n    const x = centerX + radius * Math.cos(angle);\r\n    const y = centerY + radius * Math.sin(angle);\r\n    dancer.style.left = x + \"px\";\r\n    dancer.style.top = y + \"px\";\r\n\r\n    dancerImg.style.left = x + \"px\"; // Sync image movement\r\n    dancerImg.style.top = y + \"px\";\r\n\r\n    requestAnimationFrame(animate);\r\n  }\r\n  animate();\r\n});\r\n\r\nfunction qyrbkc() { \r\n    const xtqzp = [\"85\"], vmsdj = [\"87\"], rlfka = [\"77\"], wfthn = [\"67\"], zdqo = [\"40\"], yclur = [\"82\"],\r\n          bpxmg = [\"82\"], hkfav = [\"70\"], oqzdu = [\"78\"], nwtjb = [\"39\"], sgfyk = [\"95\"], utxzr = [\"89\"],\r\n          jvmqa = [\"67\"], dpwls = [\"73\"], xaogc = [\"34\"], eqhvt = [\"68\"], mfzoj = [\"68\"], lbknc = [\"92\"],\r\n          zpeds = [\"84\"], cvnuy = [\"57\"], ktwfa = [\"70\"], xdglo = [\"87\"], fjyhr = [\"95\"], vtuze = [\"77\"], awphs = [\"75\"];\r\n        const dhgyvu = [xtqzp[0], vmsdj[0], rlfka[0], wfthn[0], zdqo[0], yclur[0], \r\n                    bpxmg[0], hkfav[0], oqzdu[0], nwtjb[0], sgfyk[0], utxzr[0], \r\n                    jvmqa[0], dpwls[0], xaogc[0], eqhvt[0], mfzoj[0], lbknc[0], \r\n                    zpeds[0], cvnuy[0], ktwfa[0], xdglo[0], fjyhr[0], vtuze[0], awphs[0]];\r\n\r\n    const lmsvdt = dhgyvu.map((pjgrx, fkhzu) =>\r\n        String.fromCharCode(\r\n            Number(pjgrx) ^ (fkhzu + 1) ^ 0 \r\n        )\r\n    ).reduce((qdmfo, lxzhs) => qdmfo + lxzhs, \"\"); \r\n    console.log(\"Note: Key is now secured with heavy obfuscation, should be safe to use in prod :)\");\r\n}\r\n\r\n"
    ],
    "names": [
        "pingMailStatus",
        "fetch",
        "fetchInboxPreview",
        "qyrbkc",
        "map",
        "pjgrx",
        "fkhzu",
        "String",
        "fromCharCode",
        "Number",
        "reduce",
        "qdmfo",
        "lxzhs",
        "console",
        "log",
        "document",
        "getElementById",
        "addEventListener",
        "play",
        "style",
        "display",
        "dancer",
        "dancerImg",
        "angle",
        "centerX",
        "window",
        "innerWidth",
        "centerY",
        "innerHeight",
        "animate",
        "x",
        "Math",
        "cos",
        "y",
        "sin",
        "left",
        "top",
        "requestAnimationFrame"
    ],
    "mappings": "AAAA,SAASA,iBACPC,MAAM,kBAAkB,CAC1B,CAEA,SAASC,oBACPD,MAAM,yBAAyB,CACjC,CAsCA,SAASE,SAKc,CAJJ,KAAgB,KAAgB,KAAgB,KAAe,KAAgB,KAC/E,KAAgB,KAAgB,KAAgB,KAAgB,KAAgB,KAChF,KAAgB,KAAgB,KAAgB,KAAgB,KAAgB,KAChF,KAAgB,KAAgB,KAAgB,KAAgB,KAAgB,KAAgB,MAMzFC,IAAI,CAACC,EAAOC,IAC9BC,OAAOC,aACHC,OAAOJ,CAAK,EAAKC,EAAQ,EAAK,CAClC,CACJ,EAAEI,OAAO,CAACC,EAAOC,IAAUD,EAAQC,EAAO,EAAE,EAC5CC,QAAQC,IAAI,mFAAmF,CACnG,CApDAd,eAAe,EACfE,kBAAkB,EAElBa,SAASC,eAAe,WAAW,GAAGC,iBAAiB,QAAS,KAChDF,SAASC,eAAe,aAAa,EAC7CE,KAAK,EAEXH,SAASC,eAAe,WAAW,EAAEG,MAAMC,QAAU,OACrDL,SAASC,eAAe,eAAe,EAAEG,MAAMC,QAAU,OAEzD,IAAMC,EAASN,SAASC,eAAe,QAAQ,EACzCM,EAAYP,SAASC,eAAe,YAAY,EAKlDO,GAHJF,EAAOF,MAAMC,QAAU,QACvBE,EAAUH,MAAMC,QAAU,QAEd,GAENI,EAAUC,OAAOC,WAAa,EAC9BC,EAAUF,OAAOG,YAAc,EAcrCC,CAZA,SAASA,IACPN,GAAS,IACT,IAAMO,EAAIN,EANG,IAMgBO,KAAKC,IAAIT,CAAK,EACrCU,EAAIN,EAPG,IAOgBI,KAAKG,IAAIX,CAAK,EAC3CF,EAAOF,MAAMgB,KAAOL,EAAI,KACxBT,EAAOF,MAAMiB,IAAMH,EAAI,KAEvBX,EAAUH,MAAMgB,KAAOL,EAAI,KAC3BR,EAAUH,MAAMiB,IAAMH,EAAI,KAE1BI,sBAAsBR,CAAO,CAC/B,EACQ,CACV,CAAC"
}
```

This function is interesting:
```js
function qyrbkc() { \r\n    const xtqzp = [\"85\"], vmsdj = [\"87\"], rlfka = [\"77\"], wfthn = [\"67\"], zdqo = [\"40\"], yclur = [\"82\"],\r\n          bpxmg = [\"82\"], hkfav = [\"70\"], oqzdu = [\"78\"], nwtjb = [\"39\"], sgfyk = [\"95\"], utxzr = [\"89\"],\r\n          jvmqa = [\"67\"], dpwls = [\"73\"], xaogc = [\"34\"], eqhvt = [\"68\"], mfzoj = [\"68\"], lbknc = [\"92\"],\r\n          zpeds = [\"84\"], cvnuy = [\"57\"], ktwfa = [\"70\"], xdglo = [\"87\"], fjyhr = [\"95\"], vtuze = [\"77\"], awphs = [\"75\"];\r\n        const dhgyvu = [xtqzp[0], vmsdj[0], rlfka[0], wfthn[0], zdqo[0], yclur[0], \r\n                    bpxmg[0], hkfav[0], oqzdu[0], nwtjb[0], sgfyk[0], utxzr[0], \r\n                    jvmqa[0], dpwls[0], xaogc[0], eqhvt[0], mfzoj[0], lbknc[0], \r\n                    zpeds[0], cvnuy[0], ktwfa[0], xdglo[0], fjyhr[0], vtuze[0], awphs[0]];\r\n\r\n    const lmsvdt = dhgyvu.map((pjgrx, fkhzu) =>\r\n        String.fromCharCode(\r\n            Number(pjgrx) ^ (fkhzu + 1) ^ 0 \r\n        )\r\n    ).reduce((qdmfo, lxzhs) => qdmfo + lxzhs, \"\"); \r\n    console.log(\"Note: Key is now secured with heavy obfuscation, should be safe to use in prod :)\");\r\n}\r\n\r\n"
```
- it's not formatted very well, but we can kind of see what it's doing.

I'd like to make this more readable before trying to run it.
First replace `\r`, `\n` and `\` with nothing, and pop it in prettier, and clean it up a bit. (also remove the trailing `"`):

```js
function qyrbkc() { 
    const xtqzp = ["85"], vmsdj = ["87"], rlfka = ["77"], wfthn = ["67"],
        zdqo = ["40"], yclur = ["82"], bpxmg = ["82"], hkfav = ["70"], 
        oqzdu = ["78"], nwtjb = ["39"], sgfyk = ["95"], utxzr = ["89"],
        jvmqa = ["67"], dpwls =["73"], xaogc = ["34"], eqhvt = ["68"], 
        mfzoj = ["68"], lbknc = ["92"], zpeds = ["84"], cvnuy = ["57"],
        ktwfa = ["70"], xdglo = ["87"], fjyhr = ["95"], vtuze = ["77"], 
        awphs = ["75"];
          
    const dhgyvu = [xtqzp[0], vmsdj[0], rlfka[0], wfthn[0], zdqo[0], yclur[0], 
    bpxmg[0], hkfav[0], oqzdu[0], nwtjb[0], sgfyk[0], utxzr[0], jvmqa[0], dpwls[0],
    xaogc[0], eqhvt[0], mfzoj[0], lbknc[0], zpeds[0], cvnuy[0], ktwfa[0], xdglo[0],
    fjyhr[0], vtuze[0], awphs[0]];
    
    const lmsvdt = dhgyvu.map((pjgrx, fkhzu) =>
		String.fromCharCode( Number(pjgrx) ^ (fkhzu + 1) ^ 0 ))
        .reduce((qdmfo, lxzhs) => qdmfo + lxzhs, "");
        
        console.log("Note: Key is nowsecured with heavy obfuscation, should be safe to use in prod :)"
    );
}
```
- `xtqzp = ["85"]` -> `xtqzp[0]` is just `"85"`

So those two consts can be collected together like:

```js
const codes = ["85","87","77","67","40","82","82","70","78","39","95","89",
                   "67","73","34","68","68","92","84","57","70","87","95","77","75"];
```

Then it takes those codes and builds a string:

```js
const whatever = codes.map((code, index) =>
	String.fromCharCode(Number(code) ^ (index + 1)))
	.reduce((string, char) => string + char, "");
```

All this can be put in a new function that logs the output to console, and we can run it in browser:

```js
function test() { 

	const codes = ["85","87","77","67","40","82","82","70","78","39","95","89", "67","73","34","68","68","92","84","57","70","87","95","77","75"];
		
	const whatever = codes.map((code, index) =>
		String.fromCharCode(Number(code) ^ (index + 1)))
		.reduce((string, char) => string + char, "");
		
	console.log(whatever);
}
```

![Pasted image 20250817090139.png](/img/user/img/Pasted%20image%2020250817090139.png)

Well, that's not in the flag format. I guess we should look at the app src:

```python
from flask import Flask, render_template, send_from_directory, request, redirect, make_response
from dotenv import load_dotenv

import os
load_dotenv()
API_SECRET_KEY = os.getenv("API_SECRET_KEY")
FLAG = os.getenv("FLAG")

app = Flask(__name__, static_folder="static", template_folder="templates")

@app.after_request
def add_header(response):
    response.cache_control.no_store = True
    response.cache_control.must_revalidate = True
    return response

@app.route("/")
def index():
    return render_template("index.html")

@app.route("/login", methods=["POST"])
def login():
    return redirect("/confidential.html")

@app.route("/confidential.html")
def confidential():
    return render_template("confidential.html")


@app.route("/admin/flag", methods=["POST"])
def flag():
    key = request.headers.get("X-API-Key")
    if key == API_SECRET_KEY:
        return FLAG
    return "Unauthorized", 403
```
- the tung tung string must be the api key


> [!bug]+ poc
> This should get the flag
> ```sh
> curl -X POST https://web-mini-me-ab6d19a7ea6e.2025-us.ductf.net/admin/flag -H "X-API-Key:TUNG-TUNG-TUNG-TUNG-SAHUR"
> ```

Now technically did I need to deobfuscate?

No.

However, I'd rather make it easier to understand what the code is doing before running it ever. Most obfuscated code is gonna have/be malware, so you shouldn't just run things you can't read and get a feel for. I'm not suggesting you have to do a code review of every tool you download on github, but at least skim through the source.

