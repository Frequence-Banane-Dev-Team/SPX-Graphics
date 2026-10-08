
# Frequence Banane
Bonchour, SPX-GC a le repo public pour SPX-Solo. SPX solo n'inclus pas les appel API mais en fouillant dans les branches github, j'ai trouver le code en clair. Donc j'ai remodifier le code, vous pouvez voir l'historique des commits. 
La suite à été réaliser avec Claude pour un peu laisser une trace des modifs. 


This is the Frequence Banane fork of SPX Graphics Controller, maintained on the `Frequence_Banane` branch. The rest of this README, after this section, is the original SPX documentation.

### Why

Starting with version 1.4.0, SPX Graphics became "SPX Solo". Most of the HTTP API (`/api/v1/...`) was replaced by placeholders that answer `501 Not Implemented – This API endpoint is not available in SPX Solo`. Those endpoints are now reserved for the paid SPX Production / Broadcast licenses.

We need these endpoints to control graphics from external tools (Stream Deck, Companion, scripts, etc.). So we brought back the original open-source code that existed before 1.4.0.

### What was modified

**1. API endpoints restored** ([routes/routes-api-v1.js](routes/routes-api-v1.js))

None of this code is new. Every endpoint was copied from this repository's own history, commit `3bfa2c8` ("Merge pull request #155", the last 1.3.x version before 1.4.0). The same code is also on the `dev-1.3.4` and `dependabot/*` branches. The only changes are the indentation and a comment saying where the code comes from. The helper functions these endpoints use still exist in 1.4.x and haven't changed.

| Method | Endpoint | What it does |
|---|---|---|
| GET | `/api/v1/item/play/:id` | Play an item of the open rundown by its ID |
| GET | `/api/v1/item/continue/:id` | Send "continue" to an item by its ID |
| GET | `/api/v1/item/stop/:id` | Stop an item by its ID |
| GET | `/api/v1/invokeTemplateFunction?...` | Call a JavaScript function inside a playing template |
| GET | `/api/v1/invokeExtensionFunction?...` | Call a function in an SPX extension |
| POST | `/api/v1/directplayout` | Fill a template and play / continue / stop it without a rundown |
| GET | `/api/v1/directplayout` | Returns a help message (only POST does something) |
| GET | `/api/v1/controlRundownItemByID?file=Project/Rundown&item=ID&command=play` | Play / stop an item of any rundown file |
| GET | `/api/v1/getprojects` | List projects |
| GET | `/api/v1/getrundowns?project=...` | List the rundowns of a project |
| GET | `/api/v1/allrundowns` | List all projects and their rundowns |
| GET | `/api/v1/gettemplates?project=...` | List the templates of a project |
| GET | `/api/v1/getlayerstate` | Show which item is on air on each web-playout layer |
| GET | `/api/v1/rundown/get` | Return the currently loaded rundown as JSON |
| GET | `/api/v1/rundown/json?project=...&rundown=...` | Return a specific rundown as JSON |
| POST | `/api/v1/rundown/json` | Create or overwrite a rundown file |
| GET | `/api/v1/executeScript?file=...` | Run a script located in `ASSETS/scripts` |
| GET | `/api/v1/getFileList?assetsfolder=...` | List the files in a subfolder of `ASSETS` |
| POST | `/api/v1/saveCustomJSON` | Save a JSON file in `ASSETS/json` |

These endpoints were already working in 1.4.x and haven't changed: `version`, `panic`, `feedproxy`, `rundown/load`, `rundown/focusFirst|Next|Previous|Last|ByID/:id`, `rundown/stopAllLayers`, `item/play|continue|stop` (focused item).

On the built-in help page `http://<server>:5656/api/v1`, the restored endpoints no longer show as disabled.

**2. API key check disabled** ([utils/spx_auth.js](utils/spx_auth.js))

`validateIfApiKey()` and `CheckAPIKey()` now let a request through even if the `apikey` is missing or wrong. A warning is still written to the log. The changed lines are marked `//TAL false` / `//false`. This lets external tools call the API without handling a key.

> ⚠️ **Security:** with this change, **anyone who can reach the SPX server on the network can control it**, including `executeScript`, which runs files from `ASSETS/scripts`. Only use it on a trusted, closed network. To turn protection back on, put back `return [false, message];` / `return false;` in `utils/spx_auth.js`.

**3. Demo project adjusted**

`DATAROOT/MyFirstProject` (rundown and profile) was modified for our tests. For example, the item ID is `test`, there's a Frequence Banane hashtag, a LiveU Dark theme, and a chroma background in the preview. The `package-lock.json` version number was also updated to 1.4.1.

---

# SPX Graphics Controller

**Manage and control HTML graphics in live production.**

>  Readme updated 2026-05-28
<br>

## SPX in a nut shell

<img src="./static/img/spx_online.png" WIDTH="60" align="left" style="margin:0 1em 0 0"> **SPX Graphics Controller** is an application using web technologies for managing and for playing back realtime graphics such as lower thirds, titles, logos, news tickers and more for live streaming, live events, live TV broadcast and other productions.  SPX runs locally on your computer or in the cloud and works seamlessly with
[OBS](https://obsproject.com/?ref=spxgraphics.com),
[vMIX](https://www.vmix.com/?ref=spxgraphics.com),
[LiveU Studio](https://www.liveu.tv/products/produce/liveu-studio),
[ToolsOnAir](https://www.toolsonair.com/),
[CasparCG](https://github.com/CasparCG),
[Wirecast](https://www.wirecast.io/en/?ref=spxgraphics.com),
[XSplit](https://www.xsplit.com/?ref=spxgraphics.com), 
[MimoLive](https://mimolive.com/?ref=spxgraphics.com),
[TopDirector](https://www.topdirector.com/?ref=spxgraphics.com),
[Tricaster](https://www.vizrt.com/products/tricaster/?ref=spxgraphics.com),
[LiveOS](https://www.neton.live/products/liveos-production-suites/?ref=spxgraphics.com),
[Tellyo](https://www.tellyo.com/?ref=spxgraphics.com),
[Sienna](https://www.sienna-tv.com/newsite/?ref=spxgraphics.com),
[AWS Elemental](https://aws.amazon.com/media-services/elemental/?ref=spxgraphics.com),
[Panasonic Kairos](https://pro-av.panasonic.net/en/products/it_ip_platform/?ref=spxgraphics.com),
[Amagi](https://www.amagi.com/?ref=spxgraphics.com)
or any other video pipeline supporting HTML sources.

* **On-Premise** - SPX can run completely on-prem without internet connection
* **Simple but feature rich** - From manual playout to 100% automated workflows
* **File system based** - simple and robust architecture
* **Bring Your Design** - SPX does not come with a design tool - use your favorite tools
* **OGraf Compliant** - SPX supports EBU OGraf file format 
* **Layers and Outputs** - use several layers of realtime graphics and target different outputs
* **Extendable** - create custom graphics workflows with plugins and APIs
* **Full graphics ecosystem** - Please visit [spxgraphics.com](https://spxgraphics.com) 🌐


<br>

> [!IMPORTANT]
> # Full docs available at → [docs.spxgraphics.com](https://docs.spxgraphics.com)



## OPEN SOURCE VERSION

Latest version 1.4.1 (May 2026)

This repository contains the open source version of the SPX Graphics Controller. You can clone the repo and use it freely. The open source version does not contain all features or API functionalities of the commercial applications, but it can be used without any other limitations or watermarks.

See [installation instructions](https://docs.spxgraphics.com/Guides/Getting+Started/Installation).


## COMMERCIAL VERSIONS

There are commercial SPX Graphics versions for professional users with additional productivity features, API endpoints, support options, Professional Services and more.

  * **[SPX Solo](https://spxgraphics.com/software/solo)** is the open source version compiled to an easy to install software for Windows, Mac and Linux and it comes with example graphics, plugins and a launcher for easy use.

  * **[SPX Production](https://spxgraphics.com/software/production)** is targeted to professional live projects, such as events, streaming, sports and other OB-productions and has additional features for professional graphics playout.

  * **[SPX Broadcast](https://spxgraphics.com/software/broadcast)** is targeted to TV broadcasters and media companies with additional features and integration options for automated graphics workflows such as 24/7 playout and newsroom with MOS, NLE and NRCS plugins.
  
<br>

  > For more information visit SPX website to [compare SPX versions](https://spxgraphics.com/software#compare) or [contact us](https://spxgraphics.com/contact).
<br>
<br>



# Screenshots <a id="screenshots"></a>

![snapshot](screenshots/00-spx-gc-principle2.png)

|  | |
| ------ | ------ |
| ![animation](screenshots/spx-gc-ui-anim-v1-0.gif) | SPX's UI is browser based and can be operated with a mouse or keyboard. Additonal _extra controls_ can be added as _plugins_ to execute specific tasks or to trigger events in external devices. |
| ![project list](screenshots/01-spx-gc-projectlist.png) | Content is managed in _projects_. Each project can have unlimited amount of _rundowns_ and _graphics templates_. Projects and their rundowns and settings are stored in _dataroot -folder_. |
| ![controller](screenshots/05-spx-gc-controller-rundown.png) | Main Controller: rundown with few items and a local preview. Items can be edited and controlled also with keyboard shortcuts. Fullscreen viewing mode recommended. Buttons below preview are customizeable. | 
| [![intro video on Youtube](screenshots/yt_mockup.png)](https://www.youtube.com/watch?v=JAo689wOL2Q) | A 20 min intro video on Youtube to learn the core concepts of the application.  | 
| [![intro video on Youtube](screenshots/yt_mockup-showreel.png)](https://youtu.be/Ruxz4DACDT4) | A showreel of SPX Graphics.  | 



## Anatomy of an example rundown item
![anatomy-of-an-item](screenshots/anatomy-of-an-item.png)


# MIT License <a id="license"></a>
Copyright 2020-2026 Tuomo Kulomaa & [SPX Graphics](http://spxgraphics.com). This project is licensed under the terms of the MIT license.
See [LICENSE.txt](LICENSE.txt)

