https://webcontainers.io/api
API Reference
The public WebContainer API allows you to build custom applications on top of an in-browser Node.js runtime. Its main entry point is the WebContainer class.

WebContainer
The main export of this library. An instance of WebContainer represents a runtime ready to be used.

WebContainer Properties

fs: FileSystemAPI
Gives access to the underlying file system.


path: string
The default value of the PATH environment variable for processes started through spawn.


workdir: string
The full path to the working directory (see FileSystemAPI).

WebContainer Methods
▸ boot
Boots a WebContainer. Only a single instance of WebContainer can be booted concurrently (see teardown). Booting WebContainer is an expensive operation.

Signature
static boot(options: BootOptions = {}): Promise<WebContainer>

Return
Returns a WebContainer instance.

BootOptions
ts
interface BootOptions {
  coep?: 'require-corp' | 'credentialless' | 'none';
  workdirName?: string;
  forwardPreviewErrors?: boolean | 'exceptions-only';
}

coep?: 'require-corp' | 'credentialless' | 'none'
The value of the COEP header used to load your application.

Choosing 'none' will result in no cross-origin isolation headers being used. This will only work on Chromium-based browsers as long as an Origin Trial is supported.

This value is fixed the first time a WebContainer is booted, and cannot be changed in successive reboots.

workdirName?: string
Sets the folder name for the working directory of your WebContainer instance. If not provided, it will be auto-generated.

This is mostly a "cosmetic" option.

forwardPreviewErrors?: boolean | 'exceptions-only'
Configure whether errors occurring in embedded preview iframes should be forwarded to the parent page. Captured errors originate from:

Calls to console.error
Any unhandledrejection events on window
Any uncaught error events on window
If set to exceptions-only, console.errors are not forwarded.

Default value is false, so no errors are emitted.

To receive the events, you register an event handler like this:

js
webcontainerInstance.on('preview-message', (message) => {
  // process the message received from a preview
});
▸ mount
Mounts a tree of files into the filesystem. This can be specified as a FileSystemTree object or as a binary snapshot generated by @webcontainer/snapshot.

Signature
mount(tree: FileSystemTree | Uint8Array | ArrayBuffer, options?: Options): Promise<void>

Options
ts
interface Options {
  mountPoint?: string;
}
▸ on
Listens for an event. The listener is called every time the event gets emitted.

Signature

on(event: 'port' | 'error' | 'server-ready' | 'preview-message', listener: PortListener | ErrorListener | ServerReadyListener | PreviewMessageListener): () => void

Returns
Returns a function to unsubscribe from the events. Once unsubscribed, the listener will no longer be called.


Overloads

▸ on(event: 'port', listener: PortListener): () => void
Listens for port events, which are emitted when a port is opened or closed by some process.

PortListener (Function)

ts
(port: number, type: "open" | "close", url: string): void

▸ on(event: 'error', listener: ErrorListener): () => void
Listens for error events, emitted when an internal error is triggered.

ErrorListener (Function)

ts
(error: { message: string }): void

▸ on(event: 'preview-message', listener: PreviewMessageListener): () => void
Listens for preview-message events, emitted when an internal error is triggered.

PreviewMessageListener (Function)

ts
(message: PreviewMessage): void
PreviewMessage
ts
type PreviewMessage = (UncaughtExceptionMessage | UnhandledRejectionMessage | ConsoleErrorMessage) & BasePreviewMessage;

interface BasePreviewMessage {
    previewId: string;
    port: number;
    pathname: string;
    search: string;
    hash: string;
}

interface UncaughtExceptionMessage {
    type: PreviewMessageType.UncaughtException;
    message: string;
    stack: string | undefined;
}

interface UnhandledRejectionMessage {
    type: PreviewMessageType.UnhandledRejection;
    message: string;
    stack: string | undefined;
}

interface ConsoleErrorMessage {
    type: PreviewMessageType.ConsoleError;
    args: any[];
    stack: string;
}

▸ on(event: 'server-ready', listener: ServerReadyListener): () => void
Listens for server-ready events, emitted when a server was started and ready to receive traffic.

ServerReadyListener (Function)

ts
(port: number, url: string): void
▸ spawn
Spawns a process. When no args are provided, spawns a process without command-line arguments.

Signature
spawn(command: string, args: string[], options?: SpawnOptions): Promise<WebContainerProcess>

Example
With args:

js
const install = await webcontainerInstance.spawn('npm', ['i']);
Without args:

js
const install = await webcontainerInstance.spawn('yarn');
Returns
Returns a WebContainerProcess.


Overloads

▸ spawn(command: string, args: string[], options?: SpawnOptions): () => void
Spawns a process with additional arguments.

▸ spawn(command: string, options?: SpawnOptions): () => void
Spawns a process without additional arguments.

▸ export
Added in version 1.4.0.

Exports the filesystem.

Signature
export(path: string, options?: ExportOptions): Promise<Uint8Array | FileSystemTree>

Example
js
const data = await webcontainerInstance.export('dist', { format: 'zip' });

const zip = new Blob([data]);
Returns
Returns a FileSystemTree when the format is json, otherwise a Uint8Array.

▸ setPreviewScript
Added in version 1.5.0.

Configure a script to be injected inside all previews. After this function resolves, every preview iframe that is either added or reloaded will now include this extra script on all HTML responses.

Notably, existing previews won't include the script until they have been reloaded.

To reload a preview you can use reloadPreview

WARNING

This API is an advanced feature that should only be used if it is your only option. Since you can control servers running in WebContainer, it's preferable to add this code when serving the content itself.


In particular, this might break existing WebContainer features or ones that will be added later.

Signature
setPreviewScript(scriptSrc: string, options?: PreviewScriptOptions): Promise<void>

Example
js
const script = `
  console.log('Hello world!');
`;

await webcontainerInstance.setPreviewScript(script);

// now all previews will always print hello world to the console if they serve HTML
▸ teardown
Destroys the WebContainer instance, turning it unusable, and releases its resources. After this, a new WebContainer instance can be obtained by calling boot.

All entities derived from this instance (e.g. processes, the file system, etc.) also become unusable after calling this method.

Signature
teardown(): void

reloadPreview
Added in version 1.2.2.

Reload the provided iframe by sending a message to the iframe and falling back to resetting the src if the iframe didn't respond in time.

Signature

reloadPreview(preview: HTMLIFrameElement, hardRefreshTimeout?: number = 200): Promise<void>

Returns
Returns a Promise that resolves when the reload has completed.

configureAPIKey
Added in version 1.3.0.

Configure an API key to be used for commercial usage of the WebContainer API. See https://webcontainers.io/enterprise for more information.

Signature

configureAPIKey(key: string): void
This function will throw an exception if WebContainer.boot was called before configureAPIKey.

auth
The authentication API is exported under the auth namespace. It allows you to authenticate users visiting your website via StackBlitz. In order for users to be authenticated via this method, they must:

Be logged in on StackBlitz.
Belong to the organisation you used to generate your clientId for use with the WebContainer API.
Authorize your website.
Once logged in, you'll be able to install private packages that those users have access to within WebContainer.

auth Functions
▸ init
Intialize the authentication for use in WebContainer. This method should be called as soon as possible as part of the loading phase of your page. For example at the top of a module that gets loaded as soon as the page loads. This is important for multiple reasons:

If you do client side routing, and the OAuth flow is happening, then query parameters might be populated with values related to the OAuth flow. The init function removes them after they've been consumed.

If you do the authentication in popup mode, you likely want the popup to be closed as soon as the authentication completed.

Signature
init(options: AuthInitOptions): { status: 'need-auth' | 'authorized' } | AuthFailedError

This function will throw an exception if WebContainer.boot was called before auth.init.


AuthInitOptions
ts
interface AuthInitOptions {
  /**
   * StackBlitz' origin.
   *
   * @default https://stackblitz.com
   */
  editorOrigin?: string;

  /**
   * The client id for this OAuth application.
   */
  clientId: string;

  /**
   * OAuth scope. The value can be found under your `Teams Settings` > `API`.
   *
   * @see https://www.rfc-editor.org/rfc/rfc6749#section-3.3
   */
  scope: string;
}
AuthFailedError
ts
interface AuthFailedError {
  status: 'auth-failed';

  /**
   * A short description of the error.
   */
  error: string;

  /**
   * A detailed description of the error.
   */
  description: string;
}
▸ startAuthFlow
This starts the OAuth flow, redirecting the current page to the StackBlitz editor to authenticate the user unless popup is set to true in which case it's done in a popup.

startAuthFlow(options?: { popup?: boolean }): void

▸ loggedIn
Returns a promise that resolves when the user authorized your application. This promise is guaranteed to never be rejected.

If the user never authorizes or declines your application, this promise never resolves.

Signature
loggedIn(): Promise<void>

Example
ts
const instance = await WebContainer.boot();

// wait until the user is logged in
await auth.loggedIn();

// we can now fetch private packages from our organisation
await instance.spawn('npm', ['install']);
▸ logout
Logout the user and clear any credentials that were saved locally.

If ignoreRevokeError is set and the revocation failed, the locally-saved credentials are discarded nonetheless.

Signature
logout(options?: { ignoreRevokeError?: boolean }): Promise<void>

▸ on
Listens for an event. The listener is called every time the event gets emitted.

Signature

on(event: 'logged-out' | 'auth-failed', listener: () => void | (reason: { error: string, description: string }) => void): () => void

Returns
Returns a function to unsubscribe from the events. Once unsubscribed, the listener will no longer be called.


Overloads

▸ on(event: 'logged-out', listener: () => void): () => void
Listens for logged-out events, which are emitted when the credentials are revoked, meaning the user needs to re-authenticate.


▸ on(event: 'auth-failed', listener: (reason: { error: string, description: string }) => void): () => void
Listens for auth-failed events, which are emitted when the user declines authorization in another tab / popup.

The property error corresponds to a constant that your code can match against, while description is a human readable error that can be useful for development. The possible values of error are:

access_denied: The user denied the authorization request.
invalid_scope: The scope is invalid, unknown or malformed.
More might be addded in the future, which is reason behind having error typed as a string instead of an union.

DirEnt
A representation of a directory entry, see the Node.js API.

DirEnt Properties

▸ name
The name of the file or directory.

Signature
name: string | Uint8Array

DirEnt Methods
▸ isDirectory
Whether the entry is a directory.

Signature
isDirectory(): boolean

▸ isFile
Whether the entry is a file.

Signature
isFile(): boolean

FileSystemAPI
Interface to interact directly with the WebContainer file system. Modeled after fs.promises in Node.

FileSystemAPI Methods
▸ mkdir
Creates a new directory. If the directory already exists, it will throw an error.

Signature
mkdir(path: String, options?: Options): Promise<void>

Options
When recursive is set to true, it will create any missing folders in the path.

ts
interface Options {
  recursive?: boolean;
}
▸ readdir
Reads a given directory and returns an array of its files and directories.

Signature
readdir(path: string, options?: Options): Promise<Uint8Array[]> | Promise<string[]> | Promise<DirEnt<Uint8Array>[]> | Promise<DirEnt<string>[]>

Options
ts
interface Options {
  encoding?: BufferEncoding;
  withFileTypes?: boolean;
}

encoding: BufferEncoding
The encoding (see BufferEncoding) can be any one of those accepted by Buffer.


withFileTypes: boolean
When set to true, the return value is an array of Dirent objects.

▸ readFile
Reads the file at the given path. If the file does not exist, it will throw an error.

Signature
readFile(path: string, encoding?: BufferEncoding | null): Promise<Uint8Array> | Promise<string>

By default, it returns a UInt8Array. A second argument can be passed as the encoding.

Example
Without encoding:

ts
const bytes = await webcontainerInstance.fs.readFile('/package.json');
With a specified encoding:

ts
const content = await webcontainerInstance.fs.readFile('/package.json', 'utf-8');
Returns
Promise<Uint8Array> or Promise<string>

▸ rename
Rename the file from oldPath to newPath. The parent folders in newPath must all exist.

Signature
rename(oldPath: string, newPath: string): Promise<void>

Example
Renaming a file:

js
await webcontainerInstance.fs.rename('/src/index.js', '/src/main.js');
▸ rm
Deletes a file or a directory. If the path is a file, it will delete the file. If the path is a directory, a second argument is needed with option recursive set to true to delete the directory and everything inside it, including nested folders.

Signature
rm(path: string, options?: Options): Promise<void>

Options
ts
interface Options {
  force?: boolean;
  recursive?: boolean;
}

force?: boolean
When true, exceptions will be ignored if the path does not exist.

recursive?: boolean
If true, it will recursively remove directories, including nested directories.

Example
Deleting a file:

js
await webcontainerInstance.fs.rm('/src/main.js');
Deleting a directory:

js
await webcontainerInstance.fs.rm('/src', { recursive: true });
▸ writeFile
Writes a file to the given path. If the file does not exist, it will create a new one. If the file exists, it will overwrite the file.

Signature
writeFile(path: string, data: string | Uint8Array, options?: string { encoding?: null | BufferEncoding } | null): Promise<void>

Example
Default:

js
await webcontainerInstance.fs.writeFile('/src/main.js', 'console.log("Hello from WebContainers!")');
With encoding:

js
await webcontainerInstance.fs.writeFile(
  '/src/main.js',
  '\xE5\x8D\x83\xE8\x91\x89\xE5\xB8\x82\xE3\x83\x96\xE3\x83\xAB\xE3\x83\xBC\xE3\x82\xB9',
  { encoding: 'latin1' }
);
▸ watch
Watch for changes to a given file or directory.

Signature
watch(path: string, options: Options, listener: Listener): Watcher

watch(path: string, listener: Listener): Watcher

Options
ts
interface Options {
  encoding?: BufferEncoding | null;
  recursive?: boolean;
}

encoding?: BufferEncoding | null
Specifies the character encoding to be used for the filename passed to the listener. Default: 'utf8'.

recursive?: boolean
Indicates whether all subdirectories should be watched, or only the current directory. This applies when a directory is specified. Default: false.

Listener
ts
type Listener = (event: 'rename' | 'change', filename: string | Buffer) => void;
Watcher
ts
interface Watcher {
  close(): void;
}

close(): void
Stop watching for changes on the given Watcher.

Example
Watching a file:

js
webcontainerInstance.fs.watch('/src/main.js', (event) => {
    console.log(`action: ${event}`);
});
Watching a directory:

js
webcontainerInstance.fs.watch('/src', { recursive: true }, (event, filename) => {
    console.log(`file: ${filename} action: ${event}`);
});
FileSystemTree
A tree-like structure to describe the contents of a folder to be mounted.

ts
interface FileSystemTree {
  [name: string]: FileNode | SymlinkNode | DirectoryNode;
}
Also see FileNode, SymlinkNode, and DirectoryNode.

Example
js
const tree = {
  myproject: {
    directory: {
      'foo.js': {
        file: {
          contents: 'const x = 1;',
        },
      },
      'bar.js': {
        file: {
          symlink: './foo.js',
        },
      },
      '.envrc': {
        file: {
          contents: 'ENVIRONMENT=staging'
        }
      },
    },
  },
  emptyFolder: {
    directory: {}
  },
};
FileNode
ts
interface FileNode {
  file: {
    contents: string | Uint8Array;
  };
}
FileNode Properties

▸ file: { contents: string | Uint8Array }
Represents a file with contents. Also see FileSystemTree.

SymlinkNode
ts
interface SymlinkNode {
  file: {
    symlink: string;
  };
}
SymlinkNode Properties

▸ file: { symlink: string }
Represents a symlink pointing to another location. Also see FileSystemTree.

DirectoryNode
ts
interface DirectoryNode {
  directory: FileSystemTree;
}
DirectoryNode Properties

▸ directory: FileSystemTree
Represents a directory node. Also see FileSystemTree.

SpawnOptions
Options that control spawning a process.

ts
export interface SpawnOptions {
  cwd?: string;
  env?: Record<string, string | number | boolean>;
  output?: boolean;
  terminal?: { cols: number; rows: number };
}
SpawnOptions Properties

▸ cwd?: string
Current working directory for the process, relative to workdir this instance (which you can change when booting WebContainer).

By default, the working directory of the spawned process is workdir.


▸ env?: Record<string, string | number | boolean>
Environment variables to set for the process.


▸ output?: boolean
When set to false, no terminal output is sent back to the process, and the output stream will never produce any chunks.


▸ terminal?: { cols: number; rows: number }
The size of the attached terminal.

ExportOptions
Options that control exporting data.

ts
export interface ExportOptions {
  format?: 'json' | 'binary' | 'zip',
  includes?: string[];
  excludes?: string[];
}
ExportOptions Properties

▸ format?: 'json' | 'binary' | 'zip'
The format of the exported data. The json and binary format can be used as tree when calling mount.

The default value is json.


▸ includes?: string[]
Globbing patterns to include files from within excluded folders.


▸ excludes?: string[]
Globbing patterns to exclude files from the export.

PreviewScriptOptions
Options that control attributes on a script injected into previews.

ts
export interface PreviewScriptOptions {
  type?: 'module' | 'importmap';
  defer?: boolean;
  async?: boolean;
}
PreviewScriptOptions Properties

▸ type?: 'module' | 'importmap'
The type attribute to use for the script. For more information, check the MDN page on the script: type attribute.


▸ defer?: boolean
If set to true, then the defer attribute will be set on the script tag. For more information, check the MDN page on the script: defer attribute.


▸ async?: boolean
If set to true, then the async attribute will be set on the script tag. For more information, check the MDN page on the script: async attribute.

WebContainerProcess
A running process spawned in a WebContainer instance.

WebContainerProcess Properties

▸ exit: Promise<number>
A promise for the exit code of the process.


▸ input: WritableStream<string>
An input stream for the attached pseudoterminal device.


▸ output: ReadableStream<string>
A stream that receives all terminal output, including the stdout and stderr emitted by the spawned process and its descendants.

Can be disabled by setting output to false via spawn.

Signature
output: ReadableStream<string>

WebContainerProcess Methods
▸ kill
Kills the process.

Signature
kill(): void

▸ resize
Resizes the attached terminal.

Signature
resize(dimensions: { cols: number, rows: number }): void

BufferEncoding
ts
type BufferEncoding =
  | 'ascii'
  | 'utf8'
  | 'utf-8'
  | 'utf16le'
  | 'ucs2'
  | 'ucs-2'
  | 'base64'
  | 'base64url'
  | 'latin1'
  | 'binary'
  | 'hex';