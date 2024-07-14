
<!--more-->


<h2>背景</h2>
<pre><code>  $ go run src\app.go
  # github.com/pebbe/zmq4
  exec: &quot;gcc&quot;: executable file not found in %PATH%
</code></pre>
<h2>安装</h2>
<ol>
<li><p>安装 <a href='http://miru.hk/archive/ZeroMQ-4.0.4~miru1.0-x64.exe'>ZeroMQ</a>, 大约 3.9 MB</p>
<p>打开 <code>ZeroMQ-4.0.4-miru1.0-x64.exe</code>, 默认安装路径是</p>
<pre><code>C:\Program Files\ZeroMQ 4.0.4
</code></pre>
<p>注意安装的是 windows 7 以后的版本, 其他 windows 版本在 <a href='http://zeromq.org/distro:microsoft-windows'>distro:microsoft-windows</a></p>
</li>
<li><p>安装 <code>gcc</code></p>
<ol>
<li><p>使用 <a href='https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/7.1.0/threads-win32/seh/x86_64-7.1.0-release-win32-seh-rt_v5-rev0.7z'>MinGW-w64</a>, 大约 44MB.</p>
<p>下载后解压 <code>x86_64-7.1.0-release-win32-seh-rt_v5-rev0.7z</code>. 注意安装的是对应 x64 版本, 其他版本在 <a href='https://sourceforge.net/projects/mingw-w64/files/'>mingw-w64/files/</a></p>
</li>
<li><p>将以下路径添加到系统环境变量 PATH</p>
<pre><code>C:\Program Files\mingw-w64\x86_64-7.1.0-win32-seh-rt_v5-rev0\mingw64\bin
</code></pre>
</li>

</ol>
</li>
<li><p>复制头文件 <code>zmq.h</code> <code>zmq_utils.h</code> </p>
<p>从 <code>C:\Program Files\ZeroMQ 4.0.4\include</code></p>
<p>到 <code>C:\Program Files\mingw-w64\x86_64-7.1.0-win32-seh-rt_v5-rev0\mingw64\x86_64-w64-mingw32\include</code></p>
</li>
<li><p>复制 lib 文件 <code>libzmq-v120-mt-gd-4_0_4.lib</code>  </p>
<p>从 <code>C:\Program Files\ZeroMQ 4.0.4\lib</code></p>
<p>到 <code>C:\Program Files\mingw-w64\x86_64-7.1.0-win32-seh-rt_v5-rev0\mingw64\x86_64-w64-mingw32\lib</code></p>
<p>改名为 <code>zmq.lib</code></p>
</li>
<li><p>复制 dll 文件 <code>libzmq-v120-mt-gd-4_0_4.dll</code></p>
<p>从 <code>C:\Program Files\ZeroMQ 4.0.4\bin</code></p>
<p>到 <code>C:\Program Files\mingw-w64\x86_64-7.1.0-win32-seh-rt_v5-rev0\mingw64\x86_64-w64-mingw32\bin</code></p>
<p>改名为<code>libzmq.dll</code></p>
</li>
<li><p>测试 <code>go get github.com/pebbe/zmq4</code>,</p>
<p>如果没报错就是成功了.</p>
</li>
<li><p>运行程序时, 出现以下症状时, 有可能需要把 <code>libzmq-v120-mt-gd-4_0_4.dll</code> 放到 <code>app.go</code> 同级目录下面.</p>
<p>症状: build正常, run异常退出</p>
<pre><code>  $ go build app.go
  $ 
  $ go run app.go 
  exit status 3221225781
</code></pre>
</li>

</ol>
<h2>FAQ</h2>
<ol>
<li>如果出现 msvcp120d.dll 未能找到的系统错误，尝试将上述所有拷贝的名为 libzmq-v120-mt-gd-4_0_4.xxx的 文件，改为拷贝  libzmq-v120-mt-4_0_4</li>

</ol>
<ol start='2' >
<li>如果遇到找不到头文件的问题，尝试将 zmq.h和 zmq_util.h复制到zmq4 的go包的根目录中</li>

</ol>
<h2>答疑</h2>
<ol>
<li><p>为什么选择 <code>libzmq-v120-mt-gd-4_0_4.dll</code> 这个文件?</p>
<blockquote><p>参考官网, 在 windows7 sp1 以后的系统, 对应的库是 <code>libzmq-v120-mt-gd-4_0_4</code></p>
<p>Stable Release 4.0.4</p>
<ul>
<li><a href='http://miru.hk/archive/ZeroMQ-4.0.4~miru1.0-x64.exe'>x64 build for Vista, 7, 8, Windows Server 2008 R2 and newer.</a></li>
<li><a href='http://miru.hk/archive/ZeroMQ-4.0.4~miru1.0-x86.exe'>x86 build for Windows XP SP3 and newer.</a></li>

</ul>
<p>Binaries built with v90, v100, v110, v110_xp, and v120 toolkits.</p>
<p>Packages include release and debug dynamic libraries along with <a href='http://msdn.microsoft.com/en-us/library/yd4f8bd1(v=vs.71).aspx'>PDB files</a> for the debug builds, static libraries are set to appear in the next stable revision.</p>
<p>The following libraries require Windows 7 SP1, Windows Server 2008R2 or later:</p>
<figure><table>
<thead>
<tr><th>Compiler</th><th>Toolkit</th><th>Release</th><th>Debug</th></tr></thead>
<tbody><tr><td>Visual Studio 2012 U4</td><td>v110</td><td><code>libzmq-v110-mt-4_0_4</code></td><td><code>libzmq-v110-mt-gd-4_0_4</code></td></tr><tr><td>Visual Studio 2013</td><td>v120</td><td><code>libzmq-v120-mt-4_0_4</code></td><td><strong>libzmq-v120-mt-gd-4_0_4</strong></td></tr></tbody>
</table></figure>
</blockquote>
</li>
<li><p>为什么不自己编译?</p>
<blockquote><p>懒</p>
</blockquote>
</li>

</ol>
