# Nexus-III

网页端检测开关，自动打开脚本：
1:打开浏览器的开发者工具（按 F12）
2:切换到控制台（Console）
3:粘贴以下脚本并回车执行
代码：
  (function ensureVPNAlwaysOn() {
    const buttonId = 'connect-toggle-button';
  
    function isButtonEnabled(button) {
      if (!button) return false;
      const slider = button.querySelector('div');
      // 判断 translate-x 是否是靠右（开启状态）
      const classList = slider?.className || '';
      return classList.includes('translate-x-[24px]') || classList.includes('translate-x-[28px]');
    }
  
    function triggerRealClick(el) {
      const event = new MouseEvent('click', {
        view: window,
        bubbles: true,
        cancelable: true
      });
      el.dispatchEvent(event);
    }
  
    function toggleIfNeeded() {
      const button = document.getElementById(buttonId);
      if (!button) {
        console.warn('[VPN] 找不到开关按钮');
        return;
      }
  
      if (!isButtonEnabled(button)) {
        console.log('[VPN] 当前为关闭状态，尝试点击开启...');
        triggerRealClick(button);
      } else {
        console.log('[VPN] 已经是开启状态');
      }
    }
  
    function waitForDomAndStart() {
      const checkReady = setInterval(() => {
        const btn = document.getElementById(buttonId);
        if (btn) {
          console.log('[VPN] 找到按钮，开始监听开关状态...');
          clearInterval(checkReady);
          toggleIfNeeded(); // 初次尝试
          setInterval(toggleIfNeeded, 2000); // 每2秒检查一次
        }
      }, 500);
    }
  
    waitForDomAndStart();
  })();


CLI报错:
Error: nexus-network: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.39' not found (required by nexus-network)

解决办法：
1:确认已安装的 GLIBC 版本：
ldd --version
它必须是 2.39，如果不是，请继续执行这些步骤
2:安装依赖项：
sudo apt update
sudo apt install -y gawk bison gcc make wget tar
3:下载 GLIBC 2.39：
wget -c https://ftp.gnu.org/gnu/glibc/glibc-2.39.tar.gz
tar -zxvf glibc-2.39.tar.gz
cd glibc-2.39
4:创建构建目录：
mkdir glibc-build
cd glibc-build
5:构建
../configure --prefix=/opt/glibc-2.39
make -j$(nproc)
sudo make install

cd

6:不要运行 nexus-network 命令，而是运行 /opt/glibc-2.39/lib/ld-linux-x86-64.so.2 --library-path /opt/glibc-2.39/lib:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu /usr/local/bin/nexus-network,例如：
/opt/glibc-2.39/lib/ld-linux-x86-64.so.2 --library-path /opt/glibc-2.39/lib:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu /root/.nexus/bin/nexus-network register-user --wallet-address your-wallet-address

注意，在上面的命令中，我的 nexus-network 目录是 /root/.nexus/bin/nexus-network ，如果您在查找正确的 nexus 目录时遇到错误，请运行以下命令：
source ~/.bashrc
which nexus-network

7:运行
现在替换为 /root/.nexus/bin/nexus-network 输出,例如：
/root/glibc-2.39/glibc-build/elf/ld-linux-x86-64.so.2 \
  --library-path /root/glibc-2.39/glibc-build:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu \
  /root/.nexus/bin/nexus-network start --node-id your-node-id
  将 your-node-id 替换为新创建的


