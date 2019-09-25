<!--- Copyright (C) 2016-2019 Lightbend Inc. <https://www.lightbend.com> -->
# Lagom�����ѧ

����һ��JonasBon��r(������[*Reactive Microservices Architecture: Design Principles for Distributed Systems*] (https://info.lightbend.com/COLL-20XX-Reactive-Microservices-Architecture-RES-LP.html))����ʶ����Ӧʽ΢�����һЩ����Ҫ��

����Lagom���Դٽ�����Щ���ʵ����

LagomĬ����������첽��-����APIͨ����ʽ����ʹ�����ͨ�ų�Ϊһ���ĸ������Lagom API��ʹ��Akka Stream���첽IO���ܽ����첽�����䣻Java APIʹ��JDK8CompletionStage�����첽���㣻Scala APIʹ��Futures��

�봫ͳ�ļ���ʽ���ݿ���ȣ�Lagom֧�ֲַ�ʽ�־�ģʽ�����ǹ���������Ҫ�󣩻����¼�����ϵ�ṹ��ʵ�����ݳ־��ԡ�ʵ��־û���Ĭ��ģʽ�����¼�Դ��ES���������ѯ���θ��루CQRS�����������ݳ־��ԴӸ߲���Ͻ�����ʲô���¼�Դ����Ϊ���м�ֵ���־�ʵ�������Lagom���¼�Դ��ʵ�֡�

Lagom�ṩ�����ڿ���Ŀ�ĵķ���ע���ͷ������ص�ʵ�֣��Լ����ڹ���ͻ��˺ͷ������˷����ֵ��ڲ��ܵ���ע��ͷ��ַ�����������Щ���


* "*����* �ǵ��Ժ�����Ե�ǰ�ᣬ������Ҫ����߽�֮����첽ͨ�� ..."
* "*���η���*ֻ��ͨ��������Э��/API��*��ŵ*����Ϊ" �� "Ҫʹ������λ��͸�����������ǿ�Ѱַ�ġ�"
* "��Ҫ����ÿ��΢������Լ���״̬����־��Գе�ȫ�����Ρ�"

����Lagom���Դٽ�����Щ���ʵ��:

* LagomĬ����������첽�� --- ����APIͨ����ʽ����ʹ�����ͨ�ų�Ϊ��һ�ȵĸ������Lagom API��ʹ��[Akka Stream](https://akka.io/)���첽IO���ܽ����첽������; Java APIʹ��[JDK8 `CompletionStage`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)�����첽���㣻Scala APIʹ��[Futures](https://www.scala-lang.org/files/archive/api/2.12.x/scala/concurrent/Future.html)��

* �봫ͳ�ļ���ʽ���ݿ���ȣ�Lagom֧�ֲַ�ʽ�־�ģʽ�����ǹ��� --- ����Ҫ�� --- �����¼�����ϵ�ṹ��ʵ�����ݳ־��ԡ�ʵ��־û���Ĭ��ģʽ�����¼�Դ(ES)�������ѯ���θ���(CQRS)��[[�������ݳ־���|ES_CQRS]]�Ӹ߲���Ͻ�����ʲô���¼�Դ����Ϊ���м�ֵ�� [[�־�ʵ��|PersistentEntity]]������Lagom���¼�Դ��ʵ�֡�

* Lagom�ṩ�����ڿ���Ŀ�ĵķ���ע��ͷ������ص�ʵ�֣��Լ����ڹ���ͻ��˺ͷ������˷����ֵ��ڲ��ܵ� [[ע��ͷ��ַ���|ServiceDiscovery]]��������Щ���
