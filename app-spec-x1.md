kind = "AppSpec"
roles = [101, 100]
type_tags = ["devops"]

[meta]
  id = "coder-code-server"
  name = "Visual Studio Code 云端版"
  version = "1.0"
  subtitle = "基于微软 VS Code 的云端集成开发工具"

[[packages]]
  name = "coder-code-server"
  version = "3.6"

[[executors]]
  name = "coder-code-server-main"
  exec_start = """
DAEMON=/opt/coder/code-server/bin/code-server
DAEMON_ARGS="--config /opt/coder/code-server/etc/config.yaml --disable-telemetry"
NAME=/opt/coder/code-server/lib/node

if pidof $NAME; then
    exit 0
fi


if [ ! -d "/opt/coder/code-server" ]; then
  mkdir -p /opt/coder/code-server
fi

rsync -av {{.inpack_prefix_coder_code_server}}/* /opt/coder/code-server/

cd /opt/coder/code-server

if [ ! -f "etc/config.yaml" ]; then
  mkdir -p etc/
  touch etc/config.yaml
fi

/home/action/.sysinner/inagent config-merge --app-spec coder-code-server --config /opt/coder/code-server/etc/config.yaml --with-config-field cfg/coder-code-server/config_yaml

if [ ! -f "/home/action/.pip/pip.conf" ]; then
  mkdir /home/action/.pip/
  cat <<< '[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
' > /home/action/.pip/pip.conf
fi

if [ ! -f "/home/action/.local/share/code-server/User/settings.json" ]; then
  mkdir -p /home/action/.local/share/code-server/User
  cat <<< '{
  "workbench.colorTheme": "Monokai"
}' > /home/action/.local/share/code-server/User/settings.json
fi

$DAEMON $DAEMON_ARGS >> /opt/coder/code-server/var/log/code-server.log 2>&1 

exit 0"""

  priority = 8

  [executors.plan]
    on_tick = 60

[[service_ports]]
  name = "http"
  box_port = 1080

[configurator]
  name = "cfg/coder-code-server"

  [[configurator.fields]]
    name = "config_yaml"
    title = "配置"
    type = 303
    default = """bind-addr: 0.0.0.0:1080
auth: password
password: {{.cfg/coder-code-server/config_password}}
cert: false"""


  [[configurator.fields]]
    name = "config_password"
    title = "登录密码"
    type = 1
    auto_fill = "hexstr_32"

[exp_res]
  cpu_min = 10
  mem_min = 512
  vol_min = 10

[exp_deploy]
  rep_min = 1
  rep_max = 1
  sys_state = 1
  network_mode = 1
  
[[urls]]
  name = "gdoc"
  url = "https://www.sysinner.cn/gdoc/view/app-guide/coder/code-server.md"

