page-type:: [[source code]]
tags:: source code, doki-theme, VSC插件
id:: 638cac41-8d85-4105-b67f-1718801f4f78

- 工作原理和核心代码：
	- 修改VSCode的本地CSS
		- 找到本地CSS路径：
		  ```typescript
		  // 找到本地css文件路径
		  const resolveWorkbench = () => {
		    if (!isWSL()) {
		      return defaultWorkbenchDirectory;
		    }
		  
		    const usersPath = path.resolve('/mnt', 'c', 'Users');
		    const users = fs.readdirSync(usersPath);
		  
		    return users.map(user => path.resolve(usersPath, user, 'AppData',
		        'Local', 'Programs', 'Microsoft VS Code', 'resources',
		        'app', 'out', 'vs', 'workbench'))
		        .filter(path => fs.existsSync(path))
		        .find(Boolean) || defaultWorkbenchDirectory;
		  };
		  ```
		- 使用文件系统，修改CSS
		  ```typescript
		  function installEditorStyles(styles: string) {
		    fs.writeFileSync(editorCss, styles, "utf-8");
		  }
		  
		  async function installStyles(
		    stickerInstallPayload: StickerInstallPayload,
		    context: vscode.ExtensionContext,
		    cssDecorator: (assets: DokiStickers) => string
		  ): Promise<InstallStatus> {
		    if (canWrite()) {
		      try {
		        const stickersAndWallpaper = await forceUpdateSticker(context, stickerInstallPayload);
		        const stickerStyles = cssDecorator(stickersAndWallpaper);
		        installEditorStyles(stickerStyles);
		        return InstallStatus.INSTALLED;
		      } catch (e) {
		        console.error("Unable to install sticker!", e);
		        if (e instanceof NetworkError) {
		          return InstallStatus.NETWORK_FAILURE;
		        }
		      }
		    }
		    return InstallStatus.FAILURE;
		  }
		  
		  
		  export async function installStickers(
		    stickerInstallPayload: StickerInstallPayload,
		    context: vscode.ExtensionContext
		  ): Promise<InstallStatus> {
		    return installStyles(stickerInstallPayload, context, (stickersAndWallpaper) =>
		      buildCSSWithStickers(stickersAndWallpaper)
		    );
		  }
		  
		  export async function installWallPaper(
		    stickerInstallPayload: StickerInstallPayload,
		    context: vscode.ExtensionContext
		  ): Promise<InstallStatus> {
		    return installStyles(stickerInstallPayload, context, (stickersAndWallpaper) =>
		      buildCSSWithWallpaperAndBackground(stickersAndWallpaper)
		    );
		  }
		  ```
	- 提供安装主题命令
		- 下略
	- 根据config动态渲染
		- 下略