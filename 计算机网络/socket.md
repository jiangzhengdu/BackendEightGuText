网络编程的基本步骤（socket）
===

答：
TCP：
服务端：socket–>bind–>listen–>accept–>recv/send–>close;
客户端：socket–>connect–>send/recv–>close.

UDP:
服务端：socket–>bind–>recvfrom/sendto–>close;
客户端：socket–>sendto/recvfrom–>close.
