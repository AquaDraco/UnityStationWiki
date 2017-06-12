# Writing Test Scripts for MonoBehaviours

Game code can get very complicated. You may want to explore test automation to help prevent bugs, drive your design, and save time. However, writing components that are testable and also functional Unity components has its own set of challenges. 

Here are a couple of guidelines for writing test scripts:

### PlayMode vs EditMode

One of the first choices you must make when writing a test is whether it will be executed in PlayMode vs EditMode. 

**EditMode** executes in the same context as your Unity Editor, which means that you can access the scripts, prefabs, and assets in your project, but can't test in-game behaviors.

**PlayMode** is like running the test in-game. You can simulate collisions, bullets, and more. Naturally, these tests are going to take longer to execute, but it'll still be faster than if you pressed Play and manually went about testing out your changes.

You can create a new test from the project right-click menu and `Create > Testing` and either EditMode or PlayMode. An EditMode test must be placed in a project folder named "Editor".

### Structure of a Test

The Unity Test Runner supports most [NUnit Test Attributes](https://github.com/nunit/docs/wiki/Attributes) as well as custom attributes to support Unity-specific test cases.

##### [UnityTest]

This is a special type of unit test that allows you to skip frames.

```c#
[UnityTest] // Return type for UnityTest must be IEnumerator
public IEnumerator Should_Test()
{
    // first frame...
    yield return 0;
    // second frame...
}
```

#### Example

```c#
using UnityEngine;
using UnityEngine.TestTools;
using NUnit.Framework;
using System.Collections;

public class ExplodeWhenShotTest {
	ExplodeWhenShot subject;

	[SetUp]
	public void SetUp() {
		var obj = new GameObject();
		obj.AddComponent<SoundManager>();
		subject = obj.AddComponent<ExplodeWhenShot>();
	}

	[UnityTest]
	public IEnumerator Should_Destroy_Bullet() {
		var bullet = new GameObject();
		var collider = bullet.AddComponent<BoxCollider2D>();
		bullet.AddComponent<Bullet_12mm>();

		subject.SendMessage("OnTriggerEnter2D", collider);

		yield return 0;

		Assert.That(bullet == null);
	}

	[UnityTest]
	public IEnumerator Should_Destroy_Object() {
		subject.Explode(null);

		yield return 0;

		Assert.That(subject == null);
	}
}

```

### Tips/Guidelines

##### You can't instantiate a MonoBehaviour directly

Instead, create a new game object and attach the behavior to it. You'll also want to attach any dependencies that your behavior relies on.

```c#
var obj = new GameObject();
obj.AddComponent<SomeDependency>();
var subject = obj.AddComponent<ScriptToTest>();
```

##### Messages are MonoBehaviour event handlers

Your scripts will probably contain one or more [Messages](https://docs.unity3d.com/ScriptReference/MonoBehaviour.html) which you would like to put under test. These methods are not invoked directly by your code, and are instead invoked by the Unity Engine itself. You can invoke Messages with the [SendMessage](https://docs.unity3d.com/ScriptReference/Component.SendMessage.html) method.

```c#
subject.SendMessage("OnTriggerEnter2D", collider);
```

##### You can't test server-side code

Sorry, even if you invoke a server-side method from your test code, the test runner behaves like a Unity Client and will just log a warning. As a workaround, you can compile the `[Server]` attribute conditionally:

```c#
#if !ENABLE_PLAYMODE_TESTS_RUNNER
[Server]
#endif
public void RunOnServer() 
{
    // ...
}

// ...

[Test]
public void Should_Test_Server_Code() 
{
    var succeeded = subject.RunOnServer();
    Assert.IsTrue(succeeded);
}
```

##### Assert that an object was destroyed

When you destroy a game object it will appear as null in an equality check `it == null`, but it won't change state until the start of the next frame. Therefore, any tests that wish to assert the destruction of an object must be in PlayMode, use the `[UnityTest]` attribute, the method return type must be `IEnumerator`, and at least one frame must have passed since the destruction action before you can assert it.

```c#
	[UnityTest]
	public IEnumerator Should_Destroy_Object() {
		subject.Explode(null);

		yield return 0;

		Assert.That(subject == null);
	}
```

It may seem clever to write `Assert.IsNull(subject);` instead, but you have to write `Assert.That(subject == null);` because your object reference will not actually be null, the Unity developers chose to override the equality operator.

##### Assert that an object was instantiated

**TODO**