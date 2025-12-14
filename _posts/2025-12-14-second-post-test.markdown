---
layout: post
title:  "The second post!"
date:   2025-12-14 15:56:00 +0100
categories: meta
tags: test
commentIssueId: 2
---
This is the second post on this site and is also just a test post to see what happens when there is a little more content on the website.

Here is some glsl code:
{% highlight glsl %}
// From: https://mimosa-pudica.net/improved-oren-nayar.html
// sigma in [0, +inf]
vec3 OrenNayarFujii(vec3 albedo, float LdotV, float NdotL, float NdotV, float sigma)
{
    float s = LdotV - NdotL * NdotV;
    float t = s <= 0 ? 1 : max(NdotL, NdotV);

    float s2 = sigma * sigma;

    const float const1 = 0.5 - (2.0 / (3.0 * PI));

    float A = 1.0 / (1.0 + const1 * sigma);
    float B = A * sigma;

    return InvPI * albedo * NdotL * (A + B * (s / max(t, 1e-30)));
}
{% endhighlight %}

And some C# code:
{% highlight cs %}
// Clustering
{
    GL.PushDebugGroup(DebugSource.DebugSourceApplication, 0, -1, "Clustering");

    ProjectionData projectionData;
    projectionData.InverseProjection = proj.Inverted();
    projectionData.ViewMatrix = view;
    projectionData.GridSize = Lighting.ClusterCounts;
    projectionData.ScreenDimentions = fbSize;
    projectionData.ZNear = camera.Near;
    projectionData.ZFar = camera.Far;

    GL.NamedBufferSubData(Lighting.ProjectionDataBuffer, 0, sizeof(ProjectionData), &projectionData);

    GL.BindBufferBase(BufferTarget.ShaderStorageBuffer, 0, Lighting.ClusterBuffer);
    GL.BindBufferBase(BufferTarget.UniformBuffer, 0, Lighting.ProjectionDataBuffer);

    GL.UseProgram(Lighting.BuildClustersShader.Program);
    Vector3i groups = (Lighting.ClusterCounts + (3, 3, 1)) / (4, 4, 2);
    GL.DispatchCompute((uint)groups.X, (uint)groups.Y, (uint)groups.Z);

    GL.UseProgram(Lighting.ClusterLightsShader.Program);
    GL.BindBufferBase(BufferTarget.ShaderStorageBuffer, 1, renderData.LightDataBuffer);
    GL.BindBufferBase(BufferTarget.ShaderStorageBuffer, 2, Lighting.LightIndexListBuffer);
    GL.BindBufferBase(BufferTarget.ShaderStorageBuffer, 3, Lighting.LightGridBuffer);

    GL.ClearNamedBufferData(Lighting.VisibleLightCountBuffer, SizedInternalFormat.R32ui, PixelFormat.Red, PixelType.UnsignedInt, 0);
    GL.MemoryBarrier(MemoryBarrierMask.AtomicCounterBarrierBit);

    GL.BindBufferBase(BufferTarget.AtomicCounterBuffer, 0, Lighting.VisibleLightCountBuffer);

    groups = (Lighting.ClusterCounts + (15, 8, 1)) / (16, 9, 2);
    GL.DispatchCompute((uint)groups.X, (uint)groups.Y, (uint)groups.Z);

    GL.PopDebugGroup();
}
{% endhighlight %}

And here is an image:
![an image](https://cdn.discordapp.com/attachments/337627185248468993/1415482236500836372/image.png?ex=693ff3cf&is=693ea24f&hm=d939fcb75c9d065c0aa51e39d6b9a6dbd6167ea058c997b563e6d6f8fd66c1c4)