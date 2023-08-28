# Angular Hello World
參考官方文檔：https://angular.tw/guide/setup-local

## 🅰️安裝流程
1. node.js安裝，可參考：[[📦NPM]]
2. `npm install -g @angular/cli --no-fund`，安裝 Angular CLI
	```bash
	$ npm install -g @angular/cli --no-fund
	added 244 packages in 21s
	npm notice
	npm notice New minor version of npm available! 9.6.7 -> 9.8.1
	npm notice Changelog: <https://github.com/npm/cli/releases/tag/v9.8.1>
	npm notice Run `npm install -g npm@9.8.1` to update!
	npm notice
	```
3. `ng new <專案名>`，建立空白專案
	```bash
	$ ng new my-app
	CREATE my-app/angular.json (2700 bytes)
	CREATE my-app/package.json (1037 bytes)
	CREATE my-app/README.md (1059 bytes)
	CREATE my-app/tsconfig.json (901 bytes)
	CREATE my-app/.editorconfig (274 bytes)
	CREATE my-app/.gitignore (548 bytes)
	CREATE my-app/tsconfig.app.json (263 bytes)
	CREATE my-app/tsconfig.spec.json (273 bytes)
	CREATE my-app/.vscode/extensions.json (130 bytes)
	CREATE my-app/.vscode/launch.json (470 bytes)
	CREATE my-app/.vscode/tasks.json (938 bytes)
	CREATE my-app/src/main.ts (214 bytes)
	CREATE my-app/src/favicon.ico (948 bytes)
	CREATE my-app/src/index.html (291 bytes)
	CREATE my-app/src/styles.css (80 bytes)
	CREATE my-app/src/app/app.module.ts (314 bytes)
	CREATE my-app/src/app/app.component.html (23083 bytes)
	CREATE my-app/src/app/app.component.spec.ts (892 bytes)
	CREATE my-app/src/app/app.component.ts (210 bytes)
	CREATE my-app/src/app/app.component.css (0 bytes)
	CREATE my-app/src/assets/.gitkeep (0 bytes)
	- Installing packages (npm)...
	√ Packages installed successfully.
	    Successfully initialized git.
	```
4. `ng serve --open`，啟動專案
	```bash
	$ cd my-app/
	$ ng serve --open
	- Generating browser application bundles (phase: setup)...
	√ Browser application bundle generation complete.
	
	Initial Chunk Files   | Names         |  Raw Size
	vendor.js             | vendor        |   2.03 MB |
	polyfills.js          | polyfills     | 333.17 kB |
	styles.css, styles.js | styles        | 230.44 kB |
	main.js               | main          |  46.00 kB |
	runtime.js            | runtime       |   6.51 kB |
	
	| Initial Total |   2.63 MB
	
	Build at: 2023-08-17T01:57:15.293Z - Hash: 06226aa442c37254 - Time: 14626ms
	
	** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **
	
	
	√ Compiled successfully.
	```
5. 測試：http://localhost:4200/
	![Angular_01_HelloWorld_01_啟動成功](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%85%B0%EF%B8%8FAngular/images/Angular_01_HelloWorld_01_%E5%95%9F%E5%8B%95%E6%88%90%E5%8A%9F.png?raw=true)

## 🅰️目錄結構
![Angular_01_HelloWorld_02_目錄結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%85%B0%EF%B8%8FAngular/images/Angular_01_HelloWorld_02_%E7%9B%AE%E9%8C%84%E7%B5%90%E6%A7%8B.png?raw=true)









