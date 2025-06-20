# eXomorph - C# Language Style Guide

This document discribes orginization conventions and how C# code should be formatted and designed for Unity. See the *C# Style Guide* for everything not mentioned here.

## Asset Naming

All Unity assets should have a suffix pertaining to what type of asset it is as follows "AssetName_TYP".

C# Scripts, Prefabs, and Scenes do not need a suffix.

|     Type|Description                 |
|--------:|:---------------------------|
|     MESH|Mesh (FBX Format)           |
|    BLEND|Blender mesh project file   |
|      DIF|Diffuse Texture             |
|     NRML|Normal Map                  |
|     DISP|Bump or displacement map    |
|     SPEC|Speciular Map               |
|      AMB|Ambient Occlusion Map       |
|[TYP]_PSD|Photoshop file              |
|      MAT|Material                    |
|      SDR|Shader/Shadergraph          |
|     SSDR|Sub-Shadergraph             |
|       RT|Render Texture              |
|     ANIM|Animation                   |
|     AMTR|Animator                    |
|      SFX|Sound Effect Audio          |
|      MUS|Music Audio                 |
|VOX_[LNG]|Dialogue Audio with Language| 

## Script Layout

Unity scripts will follow a different internal class ordering than in the *C# Style Guide*. Sections of a class should be in the following order:

- Serialized private members
- Private members
- Actions or delegates
- Public properties
- Private properties
- MonoBehaviour Method Overrides (in execution order)
- Public Methods
- Methods assigned to external callbacks
- Remaining private methods

When serializing members for the inspector, divide sections by headers using the `[Header()]` attribute. Basic sections should include "Parameters", "References", and "Inputs"

## Attributes

All attributes should be on the line above which they are affecting. The only exception is the `[SerializeField]` attribute, whick is place inline before the type (where the `private` keyword would be).

```cs
[Range(0f, 1f)]
[SerializeField] float m_smoothAmount = 0f;
```
*NOTE* : Do not use `public` to show a value in the Unity inspector. Always use the `[SerializeField]` attribute with a private member instead. If the value needs to be accessed publicly via scripting, use a public property for this.

## Serializing References

References should only be serialized in the inspector if the intended instance is within the same prefab. If a prefab containing your script cannot work after dropping the prefab in the scene, it should not be Serialized.

Use the `GetComponent` and `FindObjectByType` family of methods to locate components whenever possible. If a service or instance locator is implemented in the project, use this instead.

`GetComponent` should be called in `Awake` and `FindObjectByType` should be called in `Start` whenever possible.

## Singletons

The static singleton pattern should only be used if an object persists across all scenes.

*Do Not* use the singleton pattern only for ease of access. Use `FindObjectByType` or implemented service locator instead. In the case that a sevice locator exists, it should be the only static stingleton in the project.

When a singleton is needed, use the following pattern:
```cs
public class MySingleton : MonoBehaviour
{
    static MySingleton s_instance = null;
    public static instance
    {
        get
        { 
            if (s_instance == null)
            {
                Debug.LogError("No instance of MySingleton was assigned or created");
            }
            return s_instance;
        }
    }

    private void Awake()
    {
        if (s_instance != null)
        {
            Destroy(gameObject);
            return;
        }
        
        s_instance = this;
        DontDestroyOnLoad(gameObject);
    }
}
```
## Logging and Asserting
All assertion and error reporting rules carry over from the *C# Style Guide*.

Always use `Debug.Log`, `Debug.LogWarning`, `Debug.LogError` and `Debug.Assert` to report debug information. *Do Not* use `print()`.

All GetComponent or Service/Component Locator calls must be followed by an assert or error log unless the component is listed within a `[RequrireComponent()]` attribute.

```cs
// Assert or Debug.LogError is required
public class MyClassA : MonoBehaviour
{
    Rigidbody m_rigidbody

    private void Awake()
    {
        m_rigidbody = GetComponent<Rigidbody>();
        Debug.Assert(m_rigidbody, "Rigidbody component not found for " + name);
    }

}

// Assert or Debug.LogError not required
[RequireComponent(typeof(Rigidbody))]
public class MyClassA : MonoBehaviour
{
    Rigidbody m_rigidbody

    private void Awake()
    {
        m_rigidbody = GetComponent<Rigidbody>();
    }
}
```

**Status Logging** should be able to be toggled off by a `m_showLogging` boolean in the inspector. This is to prevent flooding the logger when looking for important messages at runtime.

## Scene Objects as Prefabs
All GameObjects must be or be a part of a Prefab and must be edited within that prefab whenever possible to prevent version control conflicts

## UnityEvents

Refer to *Callbacks (Actions & Delegates)* from the *C# Style Guide* on how to format Events. The `UnityEvent` class should only be used in cases of extreme reusibility (ex. Button on-click event).

## Colliders as Triggers

Do not use any collider components as Triggers. Instead use the `Physics.OverlapShape` family of methods. When calling these functions always provide a `LayerMask` to provide optimized detection. The `LayerMask` instance should be initialized to 0 in the script and a Debug.LogWarning should be called if not set.

Debug draw these triggers via the OnDrawGizmos method override and provide a `m_showBounds` boolean to toggle on and off (avoid `OnDrawGizmosSelected`).

```cs
public MyClass : MonoBehaviour
{
    [Header("Trigger Values")]
    [SerializeField] LayerMask m_triggerMask = 0;
    [SerializeField] Vector3 m_triggerExtends = Vector3.one;
    [SerializeField] float m_triggerOffset = 0;
    [SerializeField] bool m_showTriggerBounds = false;

    private void Awake()
    {
        if (m_triggerMask == 0)
        {
            Debug.LogWarning(name + " trigger mask was not set");
        }
    }

    private void Update()
    {
        Vector3 position = transform.position
            + transform.right * m_triggerOffset.x
            + transform.up * m_triggerOffset.y
            + transform.forward * m_triggerOffset.z;

        Collider[] hits = Physics.OverlapBox(
            position,
            m_triggerExtends * 0.5f
            transform.rotation,
            m_triggerMask);

        if (hits.Length > 0)
        {
            ProcessHit(hits);
        }
    }

    private void OnDrawGizmos()
    {
        if (!m_showTriggerBounds)
            return;

        Vector3 position = transform.position
            + transform.right * m_triggerOffset.x
            + transform.up * m_triggerOffset.y
            + transform.forward * m_triggerOffset.z;

        Gizmos.color = Color.green;
        Gizmos.matrix = Matrix4x4.TRS(
            position,
            transform.rotation,
            Vector3.one);

        Gizmos.DrawWireCube(Vector3.zero, m_triggerExtends);
    }
}
```
