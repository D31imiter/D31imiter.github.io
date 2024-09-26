#### 创建远程服务

* **创建继承Remote的IRemoteObj接口**

  ```java
  package org.example;

  import java.rmi.Remote;
  import java.rmi.RemoteException;

  public interface IRemoteObj extends Remote{
      public String sayHello(String keywords) throws RemoteException;
  }
  ```
* **RemoteObjImpl是要创建的远程对象，继承IRemoteObj接口并实现其中类方法，同时继承UnicastRemoteObject类**

  ```java
  package org.example;

  import java.rmi.RemoteException;
  import java.rmi.server.UnicastRemoteObject;

  public class RemoteObjImpl extends UnicastRemoteObject implements IRemoteObj {
      public RemoteObjImpl() throws RemoteException{

      }
      @Override
      public String sayHello(String keywords) throws RemoteException{
          String upKeywords = keywords.toUpperCase();
          System.out.println(upKeywords);
          return upKeywords;
      }
  }
  ```
* **UnicastRemoteObject构造函数中调用了exportObject，port默认为0**

  ```java
      protected UnicastRemoteObject(int port) throws RemoteException
      {
          this.port = port;
          exportObject((Remote) this, port);
      }
  ```
* **exportObject是一个静态方法**

  ```java
  public static Remote exportObject(Remote obj, int port) throws RemoteException
  {
      return exportObject(obj, new UnicastServerRef(port));
  }
  ```

  * **exportObject第二个参数UnicastServerRef的构造函数**

    ```java
        public UnicastServerRef(int port) {
            super(new LiveRef(port));
            this.filter = null;
        }
    ```

    * **跟进new LiveRef(port)**
      ```java
          public LiveRef(int port) {
              this((new ObjID()), port);
          }
      ```
    * **继续跟进**
      ```java
          public LiveRef(ObjID objID, int port) {
              this(objID, TCPEndpoint.getLocalEndpoint(port), true);
          }
      ```
    * **TCPEndpoint.getLocalEndpoint(port)即，通过ip和端口创建一个用于通信的对象,在调用getLocalEndpoint的过程中如果port==0，给port随机赋值**
      ```java
          public static TCPEndpoint getLocalEndpoint(int port) {
              return getLocalEndpoint(port, null, null);
          }
      ```
  * **UnicastServerRef父类UnicastRef**

    ```java
        public UnicastRef(LiveRef liveRef) {
            ref = liveRef;
        }
    ```
* **跟进exportObject**

  ```java
      private static Remote exportObject(Remote obj, UnicastServerRef sref)
          throws RemoteException
      {
          // if obj extends UnicastRemoteObject, set its ref.
          if (obj instanceof UnicastRemoteObject) {
              ((UnicastRemoteObject) obj).ref = sref;
          }
          return sref.exportObject(obj, null, false);
      }
  ```

  **if判断：如果要创建远程服务的对象的父类是UnicastRemoteObject，将这个对象和刚创建的UnicastServerRef绑定**
* **sref即刚创建的用于通信的对象，调用sref的exportObject**

  ```java
  public Remote exportObject(Remote impl, Object data,
                                 boolean permanent)
          throws RemoteException
      {
          Class<?> implClass = impl.getClass();
          Remote stub;

          try {
              stub = Util.createProxy(implClass, getClientRef(), forceStubUse);
          } catch (IllegalArgumentException e) {
              throw new ExportException(
                  "remote object implements illegal remote interface", e);
          }
          if (stub instanceof RemoteStub) {
              setSkeleton(impl);
          }

          Target target =
              new Target(impl, this, stub, ref.getObjID(), permanent);
          ref.exportObject(target);
          hashToMethod_Map = hashToMethod_Maps.get(implClass);
          return stub;
      }
  ```

  **创建stub**

  * **createProxy,动态代理**

    ```java
    public static Remote createProxy(Class<?> implClass,RemoteRef clientRef,boolean forceStubUse)
            throws StubNotFoundException
        {
            Class<?> remoteClass;

            try {
                remoteClass = getRemoteClass(implClass);
            } catch (ClassNotFoundException ex ) {
                throw new StubNotFoundException(
                    "object does not implement a remote interface: " +
                    implClass.getName());
            }

            if (forceStubUse ||
                !(ignoreStubClasses || !stubClassExists(remoteClass)))
            {
                return createStub(remoteClass, clientRef);
            }

            final ClassLoader loader = implClass.getClassLoader();
            final Class<?>[] interfaces = getRemoteInterfaces(implClass);
            final InvocationHandler handler =
                new RemoteObjectInvocationHandler(clientRef);

            /* REMIND: private remote interfaces? */

            try {
                return AccessController.doPrivileged(new PrivilegedAction<Remote>() {
                    public Remote run() {
                        return (Remote) Proxy.newProxyInstance(loader,
                                                               interfaces,
                                                               handler);
                    }});
            } catch (IllegalArgumentException e) {
                throw new StubNotFoundException("unable to create proxy", e);
            }
        }
    ```

    **创建Target，即再对创建的用于通讯的对象做了一次封装**
* **调用ref的exportObject**

  ```java
      public void exportObject(Target target) throws RemoteException {
          ep.exportObject(target);
      }
  ```

  **ep即**

  ```java
      public LiveRef(ObjID objID, Endpoint endpoint, boolean isLocal) {
          ep = endpoint;
          id = objID;
          this.isLocal = isLocal;
      }
  ```

  **是一个TCPEndpoint**
* **调用TCPEndpoint的exportObject方法**

  ```java
      public void exportObject(Target target) throws RemoteException {
          transport.exportObject(target);
      }
  ```
* **调用transport的exportObject方法**

  ```java
  public void exportObject(Target target) throws RemoteException {
          /*
           * Ensure that a server socket is listening, and count this
           * export while synchronized to prevent the server socket from
           * being closed due to concurrent unexports.
           */
          synchronized (this) {
              listen();
              exportCount++;
          }

          /*
           * Try to add the Target to the exported object table; keep
           * counting this export (to keep server socket open) only if
           * that succeeds.
           */
          boolean ok = false;
          try {
              super.exportObject(target);
              ok = true;
          } finally {
              if (!ok) {
                  synchronized (this) {
                      decrementExportCount();
                  }
              }
          }
      }
  ```
* **listen开启监听，调用了父类的exportObject**

  ```java
      public void exportObject(Target target) throws RemoteException {
          target.setExportedTransport(this);
          ObjectTable.putTarget(target);
      }
  ```
* **ObjectTable.putTarget**

  ```java
      static void putTarget(Target target) throws ExportException {
          ObjectEndpoint oe = target.getObjectEndpoint();
          WeakRef weakImpl = target.getWeakImpl();

          if (DGCImpl.dgcLog.isLoggable(Log.VERBOSE)) {
              DGCImpl.dgcLog.log(Log.VERBOSE, "add object " + oe);
          }

          synchronized (tableLock) {
              /**
               * Do nothing if impl has already been collected (see 6597112). Check while
               * holding tableLock to ensure that Reaper cannot process weakImpl in between
               * null check and put/increment effects.
               */
              if (target.getImpl() != null) {
                  if (objTable.containsKey(oe)) {
                      throw new ExportException(
                          "internal error: ObjID already in use");
                  } else if (implTable.containsKey(weakImpl)) {
                      throw new ExportException("object already exported");
                  }

                  objTable.put(oe, target);
                  implTable.put(weakImpl, target);

                  if (!target.isPermanent()) {
                      incrementKeepAliveCount();
                  }
              }
          }
      }
  ```

  **在**

  ```java
  ObjectEndpoint oe = target.getObjectEndpoint();
  WeakRef weakImpl = target.getWeakImpl();

  objTable.put(oe, target);
  implTable.put(weakImpl, target);
  ```

  **将相关对象放入到静态的表objTable和implTable中，等待远程链接**

#### 创建注册中心

```
Registry r = LocateRegistry.createRegistry(1099);
```

* createRegistry

  ```
      public static Registry createRegistry(int port) throws RemoteException {
          return new RegistryImpl(port);
      }
  ```
* RegistryImpl

  ```
  public RegistryImpl(int port)
          throws RemoteException
      {
          if (port == Registry.REGISTRY_PORT && System.getSecurityManager() != null) {
              // grant permission for default port only.
              try {
                  AccessController.doPrivileged(new PrivilegedExceptionAction<Void>() {
                      public Void run() throws RemoteException {
                          LiveRef lref = new LiveRef(id, port);
                          setup(new UnicastServerRef(lref, RegistryImpl::registryFilter));
                          return null;
                      }
                  }, null, new SocketPermission("localhost:"+port, "listen,accept"));
              } catch (PrivilegedActionException pae) {
                  throw (RemoteException)pae.getException();
              }
          } else {
              LiveRef lref = new LiveRef(id, port);
              setup(new UnicastServerRef(lref, RegistryImpl::registryFilter));
          }
      }
  ```
* setup

  ```
  private void setup(UnicastServerRef uref)
          throws RemoteException
      {
          /* Server ref must be created and assigned before remote
           * object 'this' can be exported.
           */
          ref = uref;
          uref.exportObject(this, null, true);
      }
  ```

  接着又调用了exportObject方法，与上一步不同的是，在exportObject中进入了

  ```
  if (stub instanceof RemoteStub) {
      setSkeleton(impl);
  }
  ```

  通过setSkeleton创建了代理

#### 绑定

```
r.bind("remoteObj",remoteObj);
```

* bind

  ```
  public void bind(String name, Remote obj)
      throws RemoteException, AlreadyBoundException, AccessException
  {
      // The access check preventing remote access is done in the skeleton
      // and is not applicable to local access.
  synchronized (bindings) {
          Remote curr = bindings.get(name);
          if (curr != null)
              throw new AlreadyBoundException(name);
          bindings.put(name, obj);
      }
  }
  ```

  bindings是Hashtable，如果这个远程对象已被绑定则抛出异常，否则将这个对象放入bindings

#### 客户端请求注册中心

```
Registry registry = LocateRegistry.getRegistry("127.0.0.1",1099);
```

* getRegistry
  ```
      public static Registry getRegistry(String host, int port,
                                         RMIClientSocketFactory csf)
          throws RemoteException
      {
          Registry registry = null;

          if (port <= 0)
              port = Registry.REGISTRY_PORT;

          if (host == null || host.length() == 0) {
              // If host is blank (as returned by "file:" URL in 1.0.2 used in
              // java.rmi.Naming), try to convert to real local host name so
              // that the RegistryImpl's checkAccess will not fail.
              try {
                  host = java.net.InetAddress.getLocalHost().getHostAddress();
              } catch (Exception e) {
                  // If that failed, at least try "" (localhost) anyway...
                  host = "";
              }
          }

          /*
           * Create a proxy for the registry with the given host, port, and
           * client socket factory.  If the supplied client socket factory is
           * null, then the ref type is a UnicastRef, otherwise the ref type
           * is a UnicastRef2.  If the property
           * java.rmi.server.ignoreStubClasses is true, then the proxy
           * returned is an instance of a dynamic proxy class that implements
           * the Registry interface; otherwise the proxy returned is an
           * instance of the pregenerated stub class for RegistryImpl.
           **/
          LiveRef liveRef =
              new LiveRef(new ObjID(ObjID.REGISTRY_ID),
                          new TCPEndpoint(host, port, csf, null),
                          false);
          RemoteRef ref =
              (csf == null) ? new UnicastRef(liveRef) : new UnicastRef2(liveRef);

          return (Registry) Util.createProxy(RegistryImpl.class, ref, false);
      }
  ```
