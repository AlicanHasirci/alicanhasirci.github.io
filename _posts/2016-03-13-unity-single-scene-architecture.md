---
layout: post
title: "Single Scene Architecture on Unity"
date: 2016-3-13 21:25:28
image: '/assets/img/'
description:
tags:
- unity
- csharp
- single
- scene
- architecture
categories:
- Unity
twitter_text:
---

I've been working on a Unity project for sometime now. Since the beginning of the project i've been really meticulous with every line of code and went to extents to optimize everything. During this times of iteration and research, i've stumbled upon on a post about single scene architures. After reading, a lot on the topic my team and i thought we would give it a shot on the game we are still working on since the design document suggested that screen tranisition should involve camera tansformation(for example during a card game, when the round ends, the round result would appear on table and the camera translates to top of table and zooms in) which would impossible with multiple scenes.

Reasons for using Single Scene Architecture
-------------------------------------------

Our first complaint on multiple-scene architecture was the static loading screens. For people who are not aware of this issue, you cannot have any animation during the loading phase which makes things really dull if you do not have a good way to tackle it. I've seen some good approaches to this issue but still(pun intended)..

Another good reason for a single scene would be Mono's behaviour of memory allocation. Unity applications use two different memories during runtime. One belongs to Unity itself for objects on scene and so on, and the other is managed by Mono for scripts. The problem here is that Mono never deallocates memory. I've read some blog posts on people pooling object before hand by instantiating them beforehand to avoid allocation during action. This approach goes well with single scene architecture since everything you need is created on start of application therefore Mono can allocate maximum memory it will use, and then avoid doing it on runtime.

The only downside is that you have to find a way to manage all your screen transitions, assets and gameobjects on one scene. Which brings us to our new topic.

A single scene to rule them all?
--------------------------------

On our case, our 'Screens' were situated in different locations in a 3D room so we hade to provide camera translations between all these screens while managing their assets during this proccess. So i had to come up with a 'ScreenManager' who would provide such behaviour.

My aim was to create 'ScreenView' gameobjects and collect everything about that screen under this objects to keep everything tidy and centralize the logic on each screen. 

```csharp
public abstract class ScreenView : View {
	public abstract Vector3 Position{get;}
	public abstract Vector3 Rotation{get;}

	public abstract void ScreenWillAppear();
	public abstract void ScreenDidAppear();
	public abstract void ScreenWillDisappear();
	public abstract void ScreenDidDisappear();
}
```

This is the script that i extend on different screen. Few things to pay attention to thou. Method names are pretty much self explainatory but the two properties Position and Rotation are not. As i have written earlier our aim was to create an illusion where the user goes to different places in a room for different screens, so position and rotation property provides this information for ScreenManager.

Anyways and here for the ScreenManager:

```csharp
	[DisallowMultipleComponent]
	public class ScreenManager : View {
		private static ScreenView currentScreen;
		private static ScreenView previousScreen;

		private static ScreenManager instance;
		public AnimationCurve cameraInterpolation;
		public List<GameObject> screens;
		public Transform cameraLookAt;

		private Dictionary<Type, ScreenView> screenDictionary;

		private static ScreenTransition inProgressScreenTransition;

		protected override void Start() {
			base.Start();

			if(instance == null) {
				instance = this;
				screenDictionary = new Dictionary<Type, ScreenView>(screens.Count);
				for(int i = 0; i < screens.Count ; i++) {
					ScreenView view = screens[i].GetComponent(typeof(ScreenView)) as ScreenView;
					view.gameObject.SetActive(false);
					screenDictionary.Add(view.GetType(), view);
				}

				screens[0].SetActive(true);
				currentScreen = screens[0].GetComponent(typeof(ScreenView)) as ScreenView;
				Camera.main.transform.position = currentScreen.Position;
				Camera.main.transform.eulerAngles = currentScreen.Rotation;

				currentScreen.ScreenDidAppear();
			}
		}

		public static T GetScreen<T>() where T : ScreenView {
			ScreenView screen;
			if(instance.screenDictionary.TryGetValue(typeof(T), out screen)){
				return screen as T;
			} else {
				throw new ArgumentException("So such screen found in ScreenManager.");
			}
		}

		public static void SwitchScreen<T>(IDictionary<string, object> parameters = null, float duration = 1f, CameraTranslation cameraTranslation = null, bool animate = true, Vector3? lookAt = null, float? fov = null) where T : ScreenView {
			if(instance == null) {
				return;
			}

			ScreenView screen;
			if(instance.screenDictionary.TryGetValue(typeof(T), out screen)) {
				if (inProgressScreenTransition != null) {
					FinalizeCurrentSwitchScreen ();
				}
				inProgressScreenTransition = new ScreenTransition(currentScreen, screen);
				previousScreen = currentScreen;
				currentScreen = screen;
				instance.StartCoroutine(instance.TranslateCamera(previousScreen, currentScreen, duration, cameraTranslation, animate, lookAt, fov));
			} else {
				inProgressScreenTransition = null;
				throw new ArgumentException("There is no "+typeof(T)+" attached on ScreenManager.");
			}
		}

		private IEnumerator TranslateCamera(ScreenView previous, ScreenView current, float duration = 0f, CameraTranslation translation = null, bool animate = true, Vector3? lookAt = null, float? fov = null) {
			current.gameObject.SetActive(true);
			previous.ScreenWillDisappear();
			current.ScreenWillAppear ();

			if(animate == true) {
				IEnumerator ct;
				if(translation == null) {
					ct = DefaultCameraTranslation(previous.Position, currentScreen.Position, previous.Rotation, current.Rotation, duration, lookAt, fov);
				} else {
					ct = translation(previous.Position, currentScreen.Position, previous.Rotation, current.Rotation, duration, lookAt);
				}
				yield return StartCoroutine(ct);
			} else {
				Camera.main.transform.position = currentScreen.Position;
				Camera.main.transform.eulerAngles = current.Rotation; 
			}
			inProgressScreenTransition = null;
			previous.gameObject.SetActive(false);
			previous.ScreenDidDisappear();
			current.ScreenDidAppear();
		}

		private IEnumerator DefaultCameraTranslation(Vector3 fromPosition, Vector3 toPosition, Vector3 fromRotation, Vector3 toRotation, float duration, Vector3? lookAt = null, float? fov = null) {
			float initialFoV = Camera.main.fieldOfView;
			float elapsedTime = 0;
			if(duration > 0) {
				while (elapsedTime < duration) {
					elapsedTime += Time.deltaTime;
					float t = cameraInterpolation.Evaluate(elapsedTime / duration);

					float xRotation = Mathf.LerpAngle(fromRotation.x, toRotation.x, t);
					float yRotation = Mathf.LerpAngle(fromRotation.y, toRotation.y, t);
					float zRotation = Mathf.LerpAngle(fromRotation.z, toRotation.z, t);
					Camera.main.transform.rotation = Quaternion.Euler(xRotation, yRotation, zRotation);
					if(fov != null) {
						Camera.main.fieldOfView = Mathf.Lerp(initialFoV, fov.Value, t);
					}
					Camera.main.transform.position = Vector3.Lerp(fromPosition, toPosition, t);
					yield return new WaitForEndOfFrame();
				}
			} else {
				Camera.main.transform.rotation = Quaternion.Euler(toRotation);
				Camera.main.transform.position = toPosition;
				if(fov != null) {
					Camera.main.fieldOfView = fov.Value;
				}
			}
		}

		private static void FinalizeCurrentSwitchScreen() {
			ScreenView previous = inProgressScreenTransition.fromScreen;
			ScreenView current = inProgressScreenTransition.toScreen;

			instance.StopAllCoroutines ();
			Camera.main.transform.position = currentScreen.Position;
			Camera.main.transform.eulerAngles = current.Rotation; 
			previous.gameObject.SetActive(false);
			previous.ScreenDidDisappear();
			current.ScreenDidAppear();
			inProgressScreenTransition = null;
		}

		public static ScreenView CurrentScreen {get{return currentScreen;}}
		public static ScreenView PreviousScreen {get{return previousScreen;}}
	}

	class ScreenTransition {
		public ScreenView fromScreen;
		public ScreenView toScreen;
		public ScreenTransition(ScreenView fromScreen, ScreenView toScreen) {
			this.fromScreen = fromScreen;
			this.toScreen = toScreen;
		}
	}

	public delegate IEnumerator CameraTranslation(Vector3 fromPosition, Vector3 toPosition, Vector3 fromRotation, Vector3 toRotation, float duration, Vector3? lookAt = null, float? fov = null);
```

Here it is. Thou it is prettu much tailored to our needs guess it will give you an idea on how to approach to issue. 

To use it:

* Add your ScreenView scripts to your gameobjects and collect them under a gameobject which has ScreenManager script.
* Bind the screen you want to use to the array under ScreenManager. First Screen in array will be used as the initial screen.
* You can call 'ScreenManager.SwitchScreen<YourScreenType>()' to go to screen you would like.