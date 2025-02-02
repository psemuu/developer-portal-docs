---

title: MLNativeSurface.cs

---


# MLNativeSurface.cs









## Source code

```csharp
// %BANNER_BEGIN%
// ---------------------------------------------------------------------
// %COPYRIGHT_BEGIN%
// Copyright (c) (2018-2022) Magic Leap, Inc. All Rights Reserved.
// Use of this file is governed by the Software License Agreement, located here: https://www.magicleap.com/software-license-agreement-ml2
// Terms and conditions applicable to third-party materials accompanying this distribution may also be found in the top-level NOTICE file appearing herein.
// %COPYRIGHT_END%
// ---------------------------------------------------------------------
// %BANNER_END%

using System.Runtime.InteropServices;

namespace UnityEngine.XR.MagicLeap
{
    public partial class MLNativeSurface
    {
        public enum PixelFormat
        {
            Custom = -4,
            Translucent = -3,
            Transparent = -2,
            Opaque = -1,
            None = 0,
            Rgba8888 = 1,
            Rgbx8888 = 2,
            Rgb888 = 3,
            Rgb565 = 4,
            Bgra8888 = 5,
            RgbaFp16 = 22,
            Rgba1010102 = 43,
        }

        public delegate void OnFrameAvailableDelegate();

        public ulong Handle { get; private set; }

        public event OnFrameAvailableDelegate OnFrameAvailable = delegate { };

        private GCHandle gcHandle;
        private bool isExternallyOwned;

        public MLNativeSurface(PixelFormat pixelFormat, ushort bufferCount, uint width, uint height)
        {
            gcHandle = GCHandle.Alloc(this, GCHandleType.Weak);
            isExternallyOwned = false;
            Handle = Native.MagicLeapNativeBindings.InvalidHandle;
            NativeBindings.MLNativeSurfaceConfig config = new NativeBindings.MLNativeSurfaceConfig(pixelFormat, bufferCount, width, height);
            MLResult.Code resultCode = NativeBindings.MLNativeSurfaceCreate(ref config, out ulong nativeSurfaceHandle);
            if (MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceCreate)))
            {
                Handle = nativeSurfaceHandle;

                NativeBindings.MLNativeSurfaceOnFrameAvailableCallback frameInfo = NativeBindings.MLNativeSurfaceOnFrameAvailableCallback.Create();
                resultCode = NativeBindings.MLNativeSurfaceSetOnFrameAvailableCallback(Handle, ref frameInfo, GCHandle.ToIntPtr(gcHandle));
                MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceSetOnFrameAvailableCallback));
            }
        }

        public MLNativeSurface(ulong surfaceHandle)
        {
            Handle = surfaceHandle;
            isExternallyOwned = true;
        }

        ~MLNativeSurface()
        {
            Destroy();
        }

        public MLResult Destroy()
        {
            MLResult.Code resultCode = MLResult.Code.NotImplemented;

            if (!isExternallyOwned && Native.MagicLeapNativeBindings.MLHandleIsValid(Handle))
            {
                resultCode = NativeBindings.MLNativeSurfaceRelease(Handle);
                MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceRelease));
                Handle = Native.MagicLeapNativeBindings.InvalidHandle;
            }
            return MLResult.Create(resultCode);
        }

        public MLResult AcquireNextAvailableFrame(out ulong nativeBuffer)
        {
            MLResult.Code resultCode = NativeBindings.MLNativeSurfaceAcquireNextAvailableFrame(Handle, out nativeBuffer);
            MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceAcquireNextAvailableFrame));
            return MLResult.Create(resultCode);
        }

        public MLResult ReleaseFrame(ulong nativeBuffer)
        {
            MLResult.Code resultCode = NativeBindings.MLNativeSurfaceReleaseFrame(Handle, nativeBuffer);
            MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceReleaseFrame));
            return MLResult.Create(resultCode);
        }

        public MLResult GetFrameTransformMatrix(float[] frameTransformMatColMajor)
        {
            MLResult.Code resultCode = NativeBindings.MLNativeSurfaceGetFrameTransformationMatrix(Handle, frameTransformMatColMajor);
            MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceGetFrameTransformationMatrix));
            return MLResult.Create(resultCode);
        }

        public MLResult GetFrameTimestamp(out long timestampNs)
        {
            MLResult.Code resultCode = NativeBindings.MLNativeSurfaceGetFrameTimestamp(Handle, out timestampNs);
            MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceGetFrameTimestamp));
            return MLResult.Create(resultCode);
        }

        public MLResult GetFrameQueueBufferTimestamp(out long timestampNs)
        {
            MLResult.Code resultCode = NativeBindings.MLNativeSurfaceGetFrameQueueBufferTimestamp(Handle, out timestampNs);
            MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceGetFrameQueueBufferTimestamp));
            return MLResult.Create(resultCode);
        }

        public MLResult GetFrameNumber(out ulong number)
        {
            MLResult.Code resultCode = NativeBindings.MLNativeSurfaceGetFrameNumber(Handle, out number);
            MLResult.DidNativeCallSucceed(resultCode, nameof(NativeBindings.MLNativeSurfaceGetFrameNumber));
            return MLResult.Create(resultCode);
        }

        private void OnFrameAvailable_CallbackThread()
        {
            OnFrameAvailable.Invoke();
        }
    }
}
```




