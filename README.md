
声明：
FreeSWITCH-EventCool 是seven du的工作成果，出于学习FreeSWITCH 的目的，对其进行了升级工作！



1：freeswitch.erl  freeswitch_event.erl 都必须要先编译，然后mv到  cb project 下的ebin中

可以通过
./init-dev.sh 启动shell后，在shell中测试是否可用  输入 fr   tab 是否后提示有freeswitch module信息


2：在freeswitch.erl 中修改 get_event_name 方法中 50行
将字符串形式修改为 二进制形式，不然会提示找不到，{error, notfound},原因可能是因为freeswitch的版本更新的原因，直接字符串找不到。
%% @doc Return the name of the event.
get_event_name(Event) ->
     get_event_header(Event, <<"Event-Name">>).

可以从freeswitch返回的事件中求证。
 {event,[undefined,
                  {<<"Event-Name">>,<<"HEARTBEAT">>},
                  {<<"Core-UUID">>,<<"960d9d56-93c0-4ef7-a7f1-69ca993958a8">>},
                  {<<"FreeSWITCH-Hostname">>,<<"localhost">>},
                  {<<"FreeSWITCH-Switchname">>,<<"localhost">>},
                  {<<"FreeSWITCH-IPv4">>,<<"192.168.23.3">>},
                  {<<"FreeSWITCH-IPv6">>,<<"::1">>},
                  {<<"Event-Date-Local">>,<<"2014-11-14 23:10:12">>},
                  {<<"Event-Date-GMT">>,<<"Fri, 14 Nov 2014 15:10:12 GMT">>},
                  {<<"Event-Date-Timestamp">>,<<"1415977812049301">>},
                  {<<"Event-Calling-File">>,<<"switch_core.c">>},
                  {<<"Event-Calling-Function">>,<<"send_heartbeat">>},
                  {<<"Event-Calling-Line-Number">>,<<"69">>},
                  {<<"Event-Sequence">>,<<"1103">>},
                  {<<"Event-Info">>,<<"System Ready">>},
                  {<<"Up-Time">>,
                   <<"0 years, 0 days, 0 hours, 22 minutes, 19 seconds, 87 milliseconds, 751 microseconds">>},
                  {<<"FreeSWITCH-Version">>,
                  
3：在启动 cb project 之后，别忘了在shell中， 调用freeswitch_event:init([]). inbound mode 注册到 freeswitch的实例上
   注册成功会有提示Connected! Register events to FreeSWITCH …
也可以通过在freeswitch console 中验证
输入  erlang listeners
输出
Listener to eventcool@localhost with outbound sessions: 0 events: 517 (lost:0) logs: 0 (lost:0) 6/6
Listener to fs_eventcool@localhost with outbound sessions: 0 events: 1018 (lost:0) logs: 0 (lost:0) 12/12


4：flush().  是个有用的函数对于erlang shell来讲


5：在cb project 的boss.config 中配置 eel 的sname and cookie  
{vm_cookie, "ClueCon"},
   {vm_sname, "fs_eventcool”},

./init-dev.sh 启动shell后，可以通过erlang:get_cookie(). 验证是否与freeswitch erlang node 的cookie 一至，这是inbound模式的关键

6：还要在cb project 的boss.config 中配置  freeswitch node 的name
[{boss, [
    {path, "./deps/boss"},
    {applications, [weibao_wiki,cb_admin]},
    {assume_locale, "en"},
    {freeswitch_node, 'freeswitch@localhost'},

7：其他的就是要注意事项就是FreeSWITCH-EventCool 是按照chicagoboss 的老版本写的，现在新版本的mvc 结构与命名方式有变化，
还有boss_db 的api方法
例如： controller 的命名方式变化为  appname_actionname_controller.erl
所有的mvc结构都放在./src/ ; 
所有的依赖都放在 ./deps/ ;
cb_admin 与你的 cb project 都在一个目录下
并且要在cb project 的boss.config 下配置
[{boss, [
    {path, "./deps/boss"},
    {applications, [weibao_wiki,cb_admin]},

。。。。。
,
 {cb_admin, [
        {path, "./deps/cb_admin"},
        {allow_ip_blocks, ["127.0.0.1"]},
        {base_url, "/admin"}
    ]}
].

还有要在rebar.config下也要配置
{deps, [
  {boss, ".*", {git, "git://github.com/ChicagoBoss/ChicagoBoss.git", {tag, "v0.8.12"}}}
  ,{cb_admin, ".*", {git, "git://github.com/ChicagoBoss/cb_admin.git", "HEAD"}}
]}.

have fun

感谢 seven du 的卓越工作以及不厌其烦的指导！