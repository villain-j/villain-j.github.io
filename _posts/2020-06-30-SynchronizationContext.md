---
title: "SynchronizationContext in C#"
categories:
  - csharp
tags:
  - CSharp
  - AsyncProgramming
  - SynchronizationContext
author_profile: true
read_time: true

classes: wide
toc: false
---

 

 

> 원본 출처: [Parallel Computing - It's All About the SynchronizationContext](https://docs.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext)

 

C#을 쓰면서 처음 본 SynchronizationContext에 대해 찾아보다가 MSDN 글이 한글 버젼이 없길래... 9년 전 글이지만 번역 반 가볍게 정리해놓는 마음 반으로 쓰는 글.

------

  

SynchronizationContext는 멀티 스레딩을 조금 더 간편하게 쓸 수 있도록 .NET 프레임워크에서 제공하는 도구이다. ASP.NET, Windows Forms, Windows Presentation Foundation (WPF), Silverlight 등등 모든 .NET 프로그램에서 사용할 수 있는 유용한 도구이다.

 

 

## SynchronizationContext가 필요한 이유

멀티스레딩 프로그램은 .NET 프레임워크 전부터 존재했었다. 보통 멀티스레딩 프로그램은 다른 스레드로 현재 스레드가 처리한 일을 넘기도록 구현할 필요가 있다. 기존 Windows 프로그램은 메시지 루프를 중심으로 돌아갔기 때문에 많은 프로그래들은 작업을 전달하기 위해 빌트인 큐를 사용해야 했다. Windows 메시지 큐를 쓰고 싶은 멀티스레딩 프로그램은 각자의 Windows 메시지를 커스텀하고 이를 처리하는 컨벤션을 만들어야 했다.

.NET 프레임워크가 처음으로 공개되면서, 이 패턴은 표준화되었다. 당시 .NET이 지원하는 유일한 GUI 어플리케이션은 Windows Forms이었다. 그러나 프레임워크 설계자들은 다른 모델을 대비해 보다 제네릭한 방법인 ISynchronizeInvoke라는 것을 만들었다.

ISynchronizeInvoke의 기본적인 아이디어는 source 스레드가 target 스레드에게 델리게이트를 큐잉시키고, 선택적으로 해당 델리게이트가 완료될 때까지 기다히는 것이다. 또한 현재 코드가 이미 target 스레드에서 실행되고 있는지를 확인할 수 있는 방법도 제공해 불필요하게 델리게이트가 큐잉되는 것 또한 막을 수 있었다. Windows Forms가 ISynchronizeInvoke의 구현을 제공했고, 비동기 컴포넌트를 설계하는 패턴이 만들어졌다. 모두가 행복:)

.NET 프레임워크 2.0에는 많은 변화가 있었다. ASP.NET 아키텍쳐에 비동기 페이지를 지원하기 시작한 것도 이 때이다. .NET 프레임워크 2.0전에는, 모든 ASP.NET의 요청은 각 요청이 완료될 때까지 스레드 하나가 필요했다. 이는 굉장히 비효율적이었는데, 웹 페이지 생성은 대부분 DB 쿼리와 웹 서비스 호출이 큰 부분이었기 떄문에 해당 요청을 처리하는 스레드는 각각의 오퍼레이션이 완료될 때까지 기다려야 했다. 비동기 페이지가 생긴 후로, 요청을 관리하는 스레드는 연산을 시작해놓고 다시 ASP.NET의 스레드 풀로 돌아가 쓰일 수 있게 되었다. 연산이 끝나면, 스레드 풀의 다른 스레드가 해당 요청을 완료 처리할 수 있었다.

그러나, ISynchronizeInvoke는 ASP.NET의 비동기 페이지 아키텍쳐에 적합하지 않았다. ASP.NET의 비동기 페이지는 하나의 스레드와 연관된 것이 아니었기 때문에 ISynchronizeInvoke 패턴으로 만들어진 비동기 컴포넌트들은 제대로 동작하지 않았다. 원래 스레드에 작업을 큐잉하는 방법 대신, 비동기 페이지는 페이지 요청이 완료되는지를 결정하기 위해 필요한 연산의 개수만 유지하기만 하면 됐다. 이 필요성에서 새롭게 설계된 SynchronizationContext가 ISynchronizeInvoke를 대신하게 되었다.

  

 

## SynchronizationContext의 개념

ISynchronizeInvoke는 두 가지 필요를 만족했다. 하나는 동기화가 필요한지 결정하는 것이었고, 다른 하나는 한 스레드에서 다른 스레드로 작업을 큐잉하는 것이었다. SynchronizationContext는 ISynchronizeInvoke를 대체하기 위해 만들어졌지만, 설계 후 기존 ISynchronizeInvoke를 정확히 대체하지는 않게 되었다.

SynchronizationContext는 작업을 context에 큐잉할 수 있는 방법을 제공해준다. 여기서 유의할 점은 작업을 특정 스레드가 아니라 context에 큐잉한다는 것이다. 아 차이는 SynchronizationContext 구현이 대부분 하나의 특정 스레드를 기준으로 하지 않는다는 점에서 매우 중요하다. 또한 SynchronizationContext는 동기화가 필요한지를 결정하는 메커니즘을 포함하지 않는데, 이를 결정하는 것이 언제나 가능한 것은 아니었기 때문이다.

SynchronizationContext의 또 다른 특징은 각 스레드가 "current" context를 갖는다는 것이다. 각 스레드의 context는 꼭 유니크하지는 앟으며, 다른 스레드와 context 인스턴스를 공유할 수 있다. 또한 스레드가 현재 자신의 context를 바꾸는 것 또한 (드물게 쓰이지만) 가능하다.

SynchronizationContext의 세 번째 특징은 현재 진행되는 비동기 연산의 개수를 가지고 있다는 것이다. 이는 ASP.NET의 비동기 페이지나 이런 종류의 개수에 관한 정보가 필요한 다른 호스트를 사용할 수 있게 해준다. 대부분의 경우에서, 현재 SynchronizationContext가 캡쳐될 때마다 count가 늘어나고, 캡쳐된 SynchronizationContext로 context에 완료를 노티할 때마다 count가 줄어든다.

SynchronizationContext에는 다른 특징들도 많지만, 이는 대부분의 프로그래머에게 중요하지 않은 것들이다. 가장 중요한 특징들을 Figure 1에 요약해 보았다.

 

**Figure 1** Aspects of the SynchronizationContext API

```xml
// The important aspects of the SynchronizationContext APIclass SynchronizationContext 
{
	// Dispatch work to the context.
	void Post(..); // (asynchronously)
	void Send(..); // (synchronously)
	// Keep track of the number of asynchronous operations.
 	void OperationStarted();
	void OperationCompleted();
	// Each thread has a current context.
	// If "Current" is null, then the thread's current context is
	// "new SynchronizationContext()", by convention.
	static SynchronizationContext Current { get; }
	static void SetSynchronizationContext(SynchronizationContext);
}
```

 

 

## SynchronizationContext의 구현들

SynchronizationContext의 실제 "context"는 정확히 정의되지 않는다. 각각의 프레임워크와 호스트는 각자의 context를 정의할 수 있다. 이런 다양한 구현과 한계를 이해하면 SynchronizationContext의 개념이 무엇을 보장하고 보장하지 않는지 알 수 있다. 간략하게 구현 몇 가지를 소개하겠다.

 

**WindowsFormsSynchronizationContext** (*System.Windows.Forms.dll: System.Windows.Forms*) 

Windows Forms 앱에서 사용하는 SynchronizationContext. UI 제어를 생성하는 스레드에서 사용된다. 여기서는 UI 제어를 위해 ISynchronizeInvoke 메서드를 사용하고, ISynchronizeInvoke 메서드는 내재된 Win32 메시지 루프에 델리게이트를 전달한다. WindowsFormsSynchronizationContext의 context는 하나의 UI 스레드이다. WindowsFormsSynchronizationContext에 큐잉된 델리게이트는 한 번에 하나씩만 특정 UI 스레드에 의해 큐잉된 순서대로 수행된다. 현재의 구현은 하나의 UI 스레드마다 하나의 WindowsFormsSynchronizationContext를 생성한다.

 

**DispatcherSynchronizationContext** (*WindowsBase.dll: System.Windows.Threading*)

WPF와 Silverlight 앱에서는 DispatcherSynchronizationContext를 사용한다. DispatcherSynchronizationContext는 UI 스레드의 Dispatcher에게 "보통" 수준의 우선순위로 델리게이트들을 큐잉한다. 스레드가 Dispatcher를 호출함으로써 Dispatcher loop을 시작할 때 SynchronizationContext가 설치된다. DispatcherSynchronizationContext에서의 context도 위와 마찬가지로 하나의 UI 스레드이다. 따라서 델리게이트가 수행되는 방식도 한 번에 하나씩 큐잉된 순서대로 특정 UI 스레드에 의해서 수행되는 방식으로 위와 같다. 현재 구현은 각각의 top-level window 하나당 DispatcherSynchronizationContext를 하나씩 생성한다. (같은 Dispatcher를 사용할지라도!)

 

**Default (ThreadPool) SynchronizationContext** (*mscorlib.dll: System.Threading*)

디폴트로 제공되는 SynchronizationContext는 default-constructed SynchronizationContext object이다. 스레드의 현재 SynchronizationContext가 null이라면, 이는 스레드가 디폴트 SynchronizationContext을 가지고 있음을 암시한다.

디폴트 SynchronizationContext는 자신의 비동기 델리게이트들을 스레드 풀에 큐잉하지만 동기적인 델리게이트는 호출 스레드에서 바로 수행한다. 따라서, 이 context는 Send를 부르는 스레드와 스레드 풀의 스레드들을 모두 커버한다. context는 Send를 호출하는 스레드를 "빌리고", 델리게이트가 완료될 때까지 context로 가져온다. 이 관점에서 보면 기본 context는 프로세스의 아무 스레드나 다 포함할 수 있다.

 스레드 풀의 스레드에는 코드가 ASP.NET에 의해 호스트되지 않는 한 디폴트 SynchronizationContext가 적용된다. 또한 Thread 클래스의 인스턴스들인 명시적인 자식 스레드에도 암시적으로 디폴트 SynchronizationContext가 적용된다. 따라서, UI 어플리케이션은 주로 두 종류의 synchronization context를 갖는다. UI SynchronizationContext는 UI 스레드에 적용되고, 디폴트 SynchronizationContext는 스레드 풀 스레드에 적용된다.

많은 이벤트 기준의 비동기 컴포넌트들은 디폴트 SynchronizationContext로는 예상대로 작동하지 않는다. 제일 널리 알려진 예시는 BackgroundWorker 하나가 다른 BackgroundWorker를 시작시키는 UI 어플리케이션에서의 오류이다. 각각의 BackgroundWorker는 RunWorkerAsync를 호출하는 스레드의 SynchronizationContext를 가져와 사용한다. 그렇게 가져온 context에서 RunWorkerCompleted 이벤트를 실행한다. 하나의 BackgroundWorker가 있을 경우, **Figure 2**처럼 RunWorkerAsync는 UI의 context에서 RunWorkerCompleted가 실행되게 한다.

![image: A Single BackgroundWorker in a UI Context](https://docs.microsoft.com/en-us/archive/msdn-magazine/2011/february/images/gg598924.cleary_figure2_hires(en-us%2cmsdn.10).jpg)

**Figure 2** A Single BackgroundWorker in a UI Context

  

그러나 만일 BackgroundWorker가 DoWork 핸들러에서 또 다른 BackgroundWorker를 부른다면, nested BackgroundWorker는 UI SynchronizationContext가 아닌 디폴트 SynchronizationContext를 가져오게 된다. 따라서 이 BackgroundWorker는 **Figure 3** 처럼 스레드 풀에서 RunWorkerCompleted를 실행하게 할 것이다.

![image: Nested BackgroundWorkers in a UI Context](https://docs.microsoft.com/en-us/archive/msdn-magazine/2011/february/images/gg598924.cleary_figure3_hires(en-us%2cmsdn.10).jpg)

**Figure 3** Nested BackgroundWorkers in a UI Context

 

기본적으로 콘솔 어플리케이션과 Windows 서비스의 모든 스레드는 디폴트 SynchronizationContext 만을 가진다. 이 때문에 몇 개의 event 기준의 비동기 컴포넌트가 제대로 작동하지 않는다. 이를 해결하기 위한 한 방법은 하나의 명시적인 자식 스레드를 생성하고, 해당 스레드에 SynchronizationContext를 설치해 이 context가 문제가 되는 컴포넌트들에게 context를 제공하게 하는 것이다. 이에 관한 자세한 내용은 이 기사 대신 Nito.Async library ([nitoasync.codeplex.com](http://nitoasync.codeplex.com/)) 를 참고해보자! (2011년 기사인데.. 지금도 살아있으려나..?)

 

**AspNetSynchronizationContext** (*System.Web.dll: System.Web [internal class]*) : ASP.NET을 모르겠어서... 이 문단도 번역하려다가 이상하게 번역할 거 같아서 원문 그대로 두고 일단은 패스...

The ASP.NET SynchronizationContext is installed on thread pool threads as they execute page code. When a delegate is queued to a captured AspNetSynchronizationContext, it restores the identity and culture of the original page and then executes the delegate directly. The delegate is directly invoked even if it’s “asynchronously” queued by calling Post.

Conceptually, the context of AspNetSynchronizationContext is complex. During the lifetime of an asynchronous page, the context starts with just one thread from the ASP.NET thread pool. After the asynchronous requests have started, the context doesn’t include any threads. As the asynchronous requests complete, the thread pool threads executing their completion routines enter the context. These may be the same threads that initiated the requests but more likely would be whatever threads happen to be free at the time the operations complete.

If multiple operations complete at once for the same application, AspNetSynchronizationContext will ensure that they execute one at a time. They may execute on any thread, but that thread will have the identity and culture of the original page.

One common example is a WebClient used from within an asynchronous Web page. DownloadDataAsync will capture the current SynchronizationContext and later will execute its DownloadDataCompleted event in that context. When the page begins executing, ASP.NET will allocate one of its threads to execute the code in that page. The page may invoke DownloadDataAsync and then return; ASP.NET keeps a count of the outstanding asynchronous operations, so it understands that the page isn’t complete. When the WebClient object has downloaded the requested data, it will receive notification on a thread pool thread. This thread will raise DownloadDataCompleted in the captured context. The context will stay on the same thread but will ensure the event handler runs with the correct identity and culture.

 

 

## SynchronizationContext의 구현에 대해 참고할 것

SynchronizationContext는 다양한 프레임워크에서 작동할 수 있는 컴포넌트를 작성할 방법을 제공한다. BackgroundWorker나 WebClient가 Windows Forms, WPF, Silverlight, console, ASP.NET 어플리케이션 모두에서 작동될 수 있는 컴포넌트로 좋은 예시이다. 그러나 이처럼 재사용할 수 있는 컴포넌트를 작성하기 위해서는 기억해두어야 할 것이 있다.

일반적으로, SynchronizationContext의 구현은 동일성을 비교할 수 없다. 이는 ISynchronizeInvoke.InvokeRequired를 대체할 수 있는 것이 없다는 뜻이다. 그러나 이는 치명적인 단점은 아니다. 여러 context를 다루려고 시도하는 것보다는 이미 알고 있는 context 안에서만 실행되는 코드가 훨씬 더 깔끔하고 검증하기 쉽다.

또한 SynchronizationContext의 구현이 델리게이트 수행 순서나 델리게이트의 동기화를 무조건 보장하지는 않는다. UI 기준의 SynchronizationContext 구현은 이를 만족하지만, ASP.NET의 SynchronizationContext는 동기화만 제공한다. 디폴트 SynchronizationContext는 수행 순서와 동기화 모두 보장하지 않는다.

또한 SynchronizationContext과 스레드 간의 1대 1 관계도 성립하지 않는다. WindowsFormsSynchronizationContext는 SynchronizationContext.CreateCopy를 호출하지 않는 한 1:1 매핑이 있지만 다른 구현에서는 모두 성립하지 않는다. 따라서 어떤 context 인스턴스도 특정 스레드에서 수행될 것이라고 가정하지 않는 것이 안전하다.

마지막으로, SynchronizationContext.Post 메서드는 항상 비동기이지 않다. 대부분의 구현이 Post를 비동기로 구현하기는 하지만, AspNetSynchronizationContext가 그러지 않는 대표적인 예외이다. 이는 예상치 못한 re-entrancy 이슈를 일으킬 수 있다.

**Figure 4**에서 SynchronizationContext의 다양한 구현에 대한 요약을 확인할 수 있다.

 

Figure 4 **Summary of SynchronizationContext Implementations**

|                     | **Specific Thread Used to Execute Delegates** | **Exclusive (Delegates Execute One at a Time)** | **Ordered (Delegates Execute in Queue Order)** | **Send May Invoke Delegate Directly** | **Post May Invoke Delegate Directly** |
| ------------------- | --------------------------------------------- | ----------------------------------------------- | ---------------------------------------------- | ------------------------------------- | ------------------------------------- |
| **Windows Forms**   | Yes                                           | Yes                                             | Yes                                            | If called from UI thread              | Never                                 |
| **WPF/Silverlight** | Yes                                           | Yes                                             | Yes                                            | If called from UI thread              | Never                                 |
| **Default**         | No                                            | No                                              | No                                             | Always                                | Never                                 |
| **ASP.NET**         | No                                            | Yes                                             | No                                             | Always                                | Always                                |

 

 

## AsyncOperationManager and AsyncOperation

.NET 프레임워크의 AsyncOperationManager와 AsyncOperation 클래스는 SynchronizationContext의 추상화를 감싸는 가벼운 wrapper이다. AsyncOperationManager는 처음으로 AsyncOperation을 생성할 때 현재의 SynchronizationContext를 가져온다. 만일 현재 SynchronizationContext가 null이라면 이를 디폴트 SynchronizationContext로 대체한다. AsyncOperation은 가져온 SynchronizationContext에 비동기적으로 델리게이트를 post한다.

대부분의 event-based 비동기 컴포넌트의 구현에서 AsyncOperationManger와 AsyncOperation을 사용한다. AsyncOperationManager와 AsyncOperation은 완료 시점이 정의된 비동기 연산에서 편리하게 쓰인다. 완료 시점이 정의됐다는 건 다른 말로는 비동기 연산이 한 시점에서 시작해 다른 시점에서 이벤트와 함께 끝난다는 것이다. 다른 비동기 알림은 완료 시점이 정의되지 않을 수도 있다. subscription과 같이 한 시점에서 시작해 완료되지 않고 계속 진행될 수 있다는 것이다. 이런 연산의 경우에는 SynchronizationContext를 바로 가져와서 직접 사용하면 된다.

새로운 컴포넌트는 더 이상 이벤트 기반의 비동기 패턴을 사용해서는 안된다. 비쥬얼 스튜디오의 비동기 Community Technology Preview (CTP) 에 task 기반의 비동기 패턴에 대한 문서가 있는데, 이는 SynchronizationContext를 통해 이벤트를 생성하는 대신 Task와 Task\<TResult> 객체를 사용하는 방법을 다룬다. Task 기반 API가 .NET에서 비동기 프로그래밍의 미래다! :angel:

 

 

## Examples of Library Support for SynchronizationContext

Simple components such as BackgroundWorker and WebClient are implicitly portable by themselves, hiding the SynchronizationContext capture and usage. Many libraries have a more visible use of SynchronizationContext. By exposing APIs using SynchronizationContext, libraries not only gain framework independence, they also provide an extensibility point for advanced end users.

In addition to the libraries I’ll discuss now, the current SynchronizationContext is considered to be part of the ExecutionContext. Any system that captures a thread’s ExecutionContext captures the current SynchronizationContext. When the ExecutionContext is restored, the SynchronizationContext is usually restored as well.

**Windows Communication Foundation (WCF):UseSynchronizationContext** WCF has two attributes that are used to configure server and client behavior: ServiceBehaviorAttribute and CallbackBehaviorAttribute. Both of these attributes have a Boolean property: UseSynchronizationContext. The default value of this attribute is true, which means that the current SynchronizationContext is captured when the communication channel is created, and this captured SynchronizationContext is used to queue the contract methods.

Normally, this behavior is exactly what is needed: Servers use the default SynchronizationContext, and client callbacks use the appropriate UI SynchronizationContext. However, this can cause problems when re-entrancy is desired, such as a client invoking a server method that invokes a client callback. In this and similar cases, the WCF automatic usage of SynchronizationContext may be disabled by setting UseSynchronizationContext to false.

This is just a brief description of how WCF uses SynchronizationContext. See the article “Synchronization Contexts in WCF” ([msdn.microsoft.com/magazine/cc163321](http://msdn.microsoft.com/magazine/cc163321)) in the November 2007 issue of *MSDN Magazine* for more details.

**Windows Workflow Foundation (WF): WorkflowInstance.SynchronizationContext** WF hosts originally used WorkflowSchedulerService and derived types to control how workflow activities were scheduled on threads. Part of the .NET Framework 4 upgrade included the SynchronizationContext property on the WorkflowInstance class and its derived WorkflowApplication class.

The SynchronizationContext may be set directly if the hosting process creates its own WorkflowInstance. SynchronizationContext is also used by WorkflowInvoker.InvokeAsync, which captures the current SynchronizationContext and passes it to its internal WorkflowApplication. This SynchronizationContext is then used to post the workflow completion event as well as the workflow activities.

**Task Parallel Library (TPL): TaskScheduler.FromCurrentSynchronizationContext and CancellationToken.Register** The TPL uses task objects as its units of work and executes them via a TaskScheduler. The default TaskScheduler acts like the default SynchronizationContext, queuing the tasks to the ThreadPool. There’s another TaskScheduler provided by the TPL queue that queues tasks to a SynchronizationContext. Progress reporting with UI updates may be done with a nested task, as shown in **Figure 5**.

Figure 5 Progress Reporting with UI Updates

XMLCopy

```xml
private void button1_Click(object sender, EventArgs e)
{
  // This TaskScheduler captures SynchronizationContext.Current.
  TaskScheduler taskScheduler = TaskScheduler.FromCurrentSynchronizationContext();
  // Start a new task (this uses the default TaskScheduler, 
  // so it will run on a ThreadPool thread).
  Task.Factory.StartNew(() =>
  {
    // We are running on a ThreadPool thread here.
   
    ; // Do some work.
   // Report progress to the UI.
    Task reportProgressTask = Task.Factory.StartNew(() =>
      {
        // We are running on the UI thread here.
        ; // Update the UI with our progress.
      },
      CancellationToken.None,
      TaskCreationOptions.None,
      taskScheduler);
    reportProgressTask.Wait();
  
    ; // Do more work.
  });
}
```

The CancellationToken class is used for any type of cancellation in the .NET Framework 4. To integrate with existing forms of cancellation, this class allows registering a delegate to invoke when cancellation is requested. When the delegate is registered, a SynchronizationContext may be passed. When the cancellation is requested, CancellationToken queues the delegate to the SynchronizationContext instead of executing it directly.

**Microsoft Reactive Extensions (Rx): ObserveOn,** **SubscribeOn and SynchronizationContextScheduler** Rx is a library that treats events as streams of data. The ObserveOn operator queues events through a SynchronizationContext, and the SubscribeOn operator queues the *subscriptions* to those events through a SynchronizationContext. ObserveOn is commonly used to update the UI with incoming events, and SubscribeOn is used to consume events from UI objects.

Rx also has its own way of queuing units of work: the IScheduler interface. Rx includes SynchronizationContextScheduler, an implementation of IScheduler that queues to a SynchronizationContext.

**Visual Studio Async CTP: await, ConfigureAwait,** **SwitchTo and EventProgress<T>** The Visual Studio support for asynchronous code transformations was announced at the Microsoft Professional Developers Conference 2010. By default, the current SynchronizationContext is captured at an await point, and this SynchronizationContext is used to resume after the await (more precisely, it captures the current SynchronizationContext *unless it is null*, in which case it captures the current TaskScheduler):

XMLCopy

```xml
private async void button1_Click(object sender, EventArgs e)
{
  // SynchronizationContext.Current is implicitly captured by await.
  var data = await webClient.DownloadStringTaskAsync(uri);
  // At this point, the captured SynchronizationContext was used to resume
  // execution, so we can freely update UI objects.
}
```

ConfigureAwait provides a means to avoid the default SynchronizationContext capturing behavior; passing false for the flowContext parameter prevents the SynchronizationContext from being used to resume execution after the await. There’s also an extension method on SynchronizationContext instances called SwitchTo; this allows any async method to change to a different SynchronizationContext by invoking SwitchTo and awaiting the result.

The asynchronous CTP introduces a common pattern for reporting progress from asynchronous operations: the IProgress<T> interface and its implementation EventProgress<T>. This class captures the current SynchronizationContext when it’s constructed and raises its ProgressChanged event in that context.

In addition to this support, void-returning async methods will increment the asynchronous operation count at their start and decrement it at their end. This behavior makes void-returning async methods act like top-level asynchronous operations.

## Limitations and Guarantees

이 문단 문장을 요약하면 뭐든 알아두면 좋다 ^-ㅠ 크로스 프레임워크 컴포넌트나 라이브러리에서 사용하는 SynchronizationContext가 뭘 할 수 있고 뭘 하지 못하는지 이해하면 관련 코드를 써먹기 좋으니까! 그리고 뭐든... 모르는 상태에서 그래서 이게 도대체 뭔데?! :angry: 하는 것보다 뭔지 대강 알아놓으면 마주쳤을 때 아 나 얘 알아! 하고 이해하는 과정이 좀 더 즐거우니까.

------

**Stephen Cleary** *has had an interest in multithreading ever since he first heard of the concept. He’s completed many business-critical multitasking systems for major clients including Syracuse News, R. R. Donnelley and BlueScope Steel. He regularly speaks at .NET user groups, BarCamps and Day of .NET events near his home in Northern Michigan, usually on a multithreading topic. He maintains a programming blog at [nitoprograms.com](http://nitoprograms.com/).*

*Thanks to the following technical expert for reviewing this article: **Eric Eilebrecht***