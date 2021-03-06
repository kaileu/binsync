# <img height="64px" title="binsync" alt="binsync logo" src="https://user-images.githubusercontent.com/46618410/60334970-0e479c00-999d-11e9-8d35-ce9ed160b3e0.png">

[![Build Status](https://travis-ci.com/retroplasma/binsync.svg?branch=master)](https://travis-ci.com/retroplasma/binsync)

A "deterministic vault" for storing private data in Usenet: Generate a code and a password and use it for incremental storage/retrieval of files. If you remember both code & password you can re-use it in the future; no metadata (like NZB) needed.
[More details here.](https://gist.github.com/retroplasma/264d9fed2350feb19f977575981bb914)
We can access the data using WebDAV:

<img src="https://user-images.githubusercontent.com/46618410/62356784-25d0f200-b512-11e9-9cff-a408f636f593.png">


#### Important

**THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.**

#### Status

We can have a "deterministic vault" in the Usenet by generating a storage code and choosing a password; keys will be generated from it. Data can be incrementally added to the vault and retrieved at a later time. It is encrypted using AES-256 (audit needed). If all metadata etc. is flushed to the Usenet we only need the code and the password to retrieve all data in the future, else we need the local cache DB. Parity is uploaded automatically (20 extra posts per 100 posts). No automatic re-uploads are implemented to deal with retention times though. The program works best with infrequent "flushes" and not too many small files because of the overhead.

The user interface is a WebDav server with a shell that can be used alongside browsing your files. The WebDav server supports reading and writing files and folders (append-only/[WORM](https://en.wikipedia.org/wiki/Write_once_read_many); no updates or deletion). Adding files has been tested with [Cyberduck](https://cyberduck.io/). Seeking into a file works, though the streaming speed hasn't been optimized yet aside from a simple look-ahead.

#### Usage (from source)

*Note: If you want to use a [precompiled release](https://github.com/retroplasma/binsync/releases), skip the first 3 setup steps and run `./Binsync.WebDavServer` in the extracted folder instead of `dotnet run`.*

So far only tested on macOS:

###### Setup:

1. Download dotnet core 2.1
2. Clone this repo
3. Go to `binsync/src/Binsync.WebDavServer`
4. Run `dotnet run`
5. It shows an example JSON config file that needs to be created; includes a random storage code.
6. Choose a password and put it into the JSON as well.
7. Add Usenet auth data to the JSON config (provider must support posting)
8. **Remember the storage code and the password for the future**

###### Run the WebDAV server and binsync shell:

1. Run `dotnet run /path/to/your/config.json --configuration=release`
2. If the config is correct it should log lots of lines when it runs the first time. It checks if any data has been uploaded with this code+pw before.
3. You should see this afterwards:
<img src="https://user-images.githubusercontent.com/46618410/62015170-5bd84400-b1a9-11e9-873a-c5c957750473.png" />

###### Access your files (WebDAV):

1. Connect to `http://localhost:5809/_dav` using Cyberduck or another program
2. You can use it to view your files. The folder will be empty in the beginning though.
3. You can create folders and files using Cyberduck: Write `write 1` into binsync's shell first to enable writing. Remember that writes cannot be reverted: They are append-only.

###### Flushing (binsync shell):

1. A local cache (metadata, residual parity and journal) will be saved locally according to you config. These files are important to have around before you use "`flush`". To flush to the Usenet you need to write `flush` into the shell. When it's done you should be able to restore everything using code+pw in theory. Make sure to not write and flush from multiple machines at the same time; this isn't safe. Frankly nothing of this is safe of course.
2. Use `help` in binsync's shell for more info.
