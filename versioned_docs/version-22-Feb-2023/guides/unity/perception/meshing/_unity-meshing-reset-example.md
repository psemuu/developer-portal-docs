---
id: unity-meshing-reset-example
title: Meshing Refresh Example
description: Learn to adjust the appearance of the mesh generated by the Magic Leap 2.
sidebar_position: 0
date: 06/08/2022
tags: [Unity, Perception, Meshing, Surfaces]
keywords: [Unity, Perception, Meshing, Surfaces]
---

This section includes an example on how to reset and refresh the meshes that were generated by the [`UnityEngine.XR.MagicLeap.MeshingSubsystemComponent`](/versioned_docs/version-22-Feb-2023/guides/unity/perception/meshing/unity-meshing-subsystem-component.md). Developers may wish to perform these actions when headpose is lost or if the user changes the visualization of the meshes.

:::caution

Before proceeding make sure you have an object with a properly configured `MeshingSubsystemComponent` attached. In addition to having the `WORLD_RECONSTRUCTION` permission to be enabled in your project's Manifest Settings. (**Edit > Project Settings > Magic Leap > Manifest Settings**)

:::

## Example

This example clears the generated meshes when the user resets their head pose.

```csharp showLineNumbers
using UnityEngine;
using UnityEngine.XR;
using UnityEngine.XR.MagicLeap;
using UnityEngine.XR.Management;

public class ResetMeshingExample : MonoBehaviour
{
    [SerializeField, Tooltip("The spatial mapper from which to update mesh params.")]
    private MeshingSubsystemComponent _meshingSubsystemComponent = null;

    //Used to track if the user lost head pose
    private XRInputSubsystem _inputSubsystem;

    void Start()
    {
        _inputSubsystem = XRGeneralSettings.Instance?.Manager?.activeLoader?.GetLoadedSubsystem<XRInputSubsystem>();
        if(_inputSubsystem!=null)
            _inputSubsystem.trackingOriginUpdated += OnTrackingOriginChanged;
    }

    void OnDestroy()
    {
        if(_inputSubsystem!=null) 
            _inputSubsystem.trackingOriginUpdated -= OnTrackingOriginChanged;
    }

    /// <summary>
    /// Handle in charge of refreshing all meshes if a new session occurs
    /// </summary>
    /// <param name="inputSubsystem"> The inputSubsystem that invoked this event. </param>
    private void OnTrackingOriginChanged(XRInputSubsystem inputSubsystem)
    {
#if UNITY_MAGICLEAP || UNITY_ANDROID
        _meshingSubsystemComponent.DestroyAllMeshes();
        _meshingSubsystemComponent.RefreshAllMeshes();
#endif
    }
}
```

