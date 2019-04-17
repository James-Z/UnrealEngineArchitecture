Unreal Engine Architecture Guide
================================
Classes
-------
* ### FSceneRender
* ### FScene
* ### FPrimitiveSceneInfo
* ### FMeshDrawCommand
* ### FCachedPassMeshDrawList
* ### FMeshDrawCommand
* ### FVisibleMeshDrawCommand
* ### FMeshPassDrawListContext
* ### FMeshPassProcessor
* ### FBasePassMeshProcessor
* ### FCachedMeshDrawCommandInfo
* ### FPrimitiveSceneInfo
    * [StaticMeshCommandInfos](#FCachedMeshDrawCommandInfo)
        >Store the draw commands into the cache of FPrimitiveSceneInfo
* ### FVisibleLightInfo
* ### FSceneViewFamily
* ### FViewInfo
* ### FPrimitiveViewRelevance
* ### FSceneViewState
    ```c++
    /**
    * The scene manager's private implementation of persistent view state.
    * This class is associated with a particular camera across multiple frames by the game thread.
    * The game thread calls FRendererModule::AllocateViewState to create an instance of this private implementation.
    */
    class FSceneViewState : public FSceneViewStateInterface, public FDeferredCleanupInterface, public FRenderResource
    ```
* ### FLODSceneTree
* ### FRelevancePacket
* ### FRelevancePrimSet
* ### FTranslucenyPrimCount
* ### FStaticMeshBatchRelevance
* ### EMeshPass::Type
    ```c++
    /** Mesh pass types supported. */
    namespace EMeshPass
    {
    	enum Type
    	{
    		DepthPass,
    		BasePass,
    		CSMShadowDepth,
    		Distortion,
    		Velocity,
    		TranslucencyStandard,
    		TranslucencyAfterDOF,
    		TranslucencyAll, /** Drawing all translucency, regardless of separate or standard.  Used when drawing translucency outside of the main  renderer, eg FRendererModule::DrawTile. */
    		LightmapDensity,
    		DebugViewMode, /** Any of EDebugViewShaderMode */
    		CustomDepth,
    		MobileBasePassCSM,  /** Mobile base pass with CSM shading enabled */
    		MobileInverseOpacity,  /** Mobile specific scene capture, Non-cached */

    #if WITH_EDITOR
    		HitProxy,
    		HitProxyOpaqueOnly,
    		EditorSelection,
    #endif

    		Num,
    		NumBits = 5,
    	};
    }
    ```
* ### FStaticMeshBatch
    ```c++
    /**
    * A mesh which is defined by a primitive at scene segment construction time and never changed.
    * Lights are attached and detached as the segment containing the mesh is added or removed from a scene.
    */
    class FStaticMeshBatch : public FMeshBatch
    ```
* ### FViewCommands
    ```c++
    class FViewCommands
    {
    public:
    	FViewCommands()
    	{
    		for (int32 PassIndex = 0; PassIndex < EMeshPass::Num; ++PassIndex)
    		{
    			NumDynamicMeshCommandBuildRequestElements[PassIndex] = 0;
    		}
    	}

    	TStaticArray<FMeshCommandOneFrameArray, EMeshPass::Num> MeshCommands;
    	TStaticArray<int32, EMeshPass::Num> NumDynamicMeshCommandBuildRequestElements;
    	TStaticArray<TArray<const FStaticMeshBatch*, SceneRenderingAllocator>, EMeshPass::Num> DynamicMeshCommandBuildRequests;
    };
    ```
* ### FMeshElementCollector
    ```c++
    /** 
    * Encapsulates the gathering of meshes from the various FPrimitiveSceneProxy classes. 
    */
    class FMeshElementCollector
    ```
    * 
* ### FParallelMeshDrawCommandPass
    ```c++
    /**
    * Parallel mesh draw command processing and rendering. 
    * Encapsulates two parallel tasks - mesh command setup task and drawing task.
    */
    class FParallelMeshDrawCommandPass
    ```
* ### FSceneViewState
    ```c++
    /**
     * The scene manager's private implementation of persistent view state.
     * This class is associated with a particular camera across multiple frames by the game thread.
     * The game thread calls FRendererModule::AllocateViewState to create an instance of this private implementation.
    */
    class FSceneViewState : public FSceneViewStateInterface, public FDeferredCleanupInterface, public FRenderResource
    ```
* ### FPrecomputedLightVolume
    ```c++
    /** Set of volume lighting samples belonging to one streaming level, which can be queried about the lighting at a given position. */
    class FPrecomputedLightVolume
    ```
* ### FIndirectLightingCache
    ```c++
    /** 
     * Implements a volume texture atlas for caching indirect lighting on a per-object basis.
     * The indirect lighting is interpolated from Lightmass SH volume lighting samples.
     */
    class FIndirectLightingCache : public FRenderResource
    ```
* ### ICustomVisibilityQuery
    ```c++
    class ICustomVisibilityQuery: public IRefCountedObject
    {
    public:
    	/** prepares the query for visibility tests */
    	virtual bool Prepare() = 0;

    	/** test primitive visiblity */
    	virtual bool IsVisible(int32 VisibilityId, const FBoxSphereBounds& Bounds) = 0;

    	/** return true if we can call IsVisible from a ParallelFor */
    	virtual bool IsThreadsafe()
    	{
    		return false;
    	}
    };
    ```

Pipeline
--------
[InitView(...)](#InitViews(...))

Method
------
* ### InitViews(...)
    * [PreVisibilityFrameSetup(...)](#FDeferredShadingSceneRenderer::PreVisibilityFrameSetup(...))
        * Performs once per frame setup prior to visibility determination.  
        * Involve base class [FSceneRender](#FSceneRender)::[PreVisibilityFrameSetup](#FSceneRender::PreVisibilityFrameSetup(...))
        * ...TODO
    * [ComputeViewVisibility(...)](#FSceneRenderer::ComputeViewVisibility(...))
        * Primitives culled by [OcclusionCull(...)](#OcclusionCull(...)) and [FrustumCull(...)](FrustumCull(...))
        * Visiable primitives classify into different BitArray
        * Create draw commands, if static meshes find the draw command cache in CommandCache in scene, if dynamic mesh store them MeshBatch in a container
        * Create draw commands for dynamic meshes
    * [CreateIndirectCapsuleShadows()](#FDeferredShadingSceneRenderer::CreateIndirectCapsuleShadows())
    * [PostVisibilityFrameSetup(...)](#FSceneRenderer::PostVisibilityFrameSetup(...))
    * [InitViewsPossiblyAfterPrepass(...)](#FDeferredShadingSceneRenderer::InitViewsPossiblyAfterPrepass(...))
* ### FDeferredShadingSceneRenderer::InitViewsPossiblyAfterPrepass(...)
    ```c++
    void FDeferredShadingSceneRenderer::InitViewsPossiblyAfterPrepass(FRHICommandListImmediate& RHICmdList, struct FILCUpdatePrimTaskData& ILCTaskData, FGraphEventArray& UpdateViewCustomDataEvents)
    ```
    * [InitDynamicShadows(...)](#FSceneRenderer::InitDynamicShadows(...))
    * 
* ### FSceneRenderer::InitDynamicShadows(...)
     ```c++
    void FSceneRenderer::InitDynamicShadows(FRHICommandListImmediate& RHICmdList, FGlobalDynamicIndexBuffer&DynamicIndexBuffer, FGlobalDynamicVertexBuffer& DynamicVertexBuffer, FGlobalDynamicReadBuffer&DynamicReadBuffer)
    ```
    * ...TODO
* ### FSceneRenderer::PostVisibilityFrameSetup(...)
    ```c++
    void FSceneRenderer::PostVisibilityFrameSetup(FILCUpdatePrimTaskData& OutILCTaskData)
    ```
    * Sort the MeshDecalBatches in [Views](#FViewInfo)
    * Check if LightComponent is used, if not remove from [FSceneViewState](#FSceneViewState).LightShaftBloomHistoryRTs
        ```c++
        TMap<const ULightComponent*, FTemporalAAHistory > LightShaftBloomHistoryRTs;
        ```
    * Clear mobile light shaft data
    * Upadte [IndirectLightingCache](#FIndirectLightingCache).[StartUpdateCachePrimitivesTask(...)](FIndirectLightingCache::StartUpdateCachePrimitivesTask(...)) if [PrecomputedLightVolumes](#FPrecomputedLightVolume) not empty
    * Check lights determine which is visiable, check lights' VisiableBit and draw spheres intersect with Frustum overlay
    * [InitFogConstants()](#FSceneRenderer::InitFogConstants())
* ### FSceneRenderer::InitFogConstants()
    ```c++
    void FSceneRenderer::InitFogConstants()
    ```
    * ...TODO
* ### FIndirectLightingCache::StartUpdateCachePrimitivesTask(...)
    ```c++
    void FIndirectLightingCache::StartUpdateCachePrimitivesTask(FScene* Scene, FSceneRenderer& Renderer, bool bAllowUnbuiltPreview, FILCUpdatePrimTaskData& OutTaskData)
    ```
    * ...TODO
* ### FDeferredShadingSceneRenderer::CreateIndirectCapsuleShadows()
    ```c++
    void FDeferredShadingSceneRenderer::CreateIndirectCapsuleShadows()
    ```
    * ...TODO
* ### FSceneRenderer::ComputeViewVisibility(...)
    ```c++
    void FSceneRenderer::ComputeViewVisibility(FRHICommandListImmediate& RHICmdList, FExclusiveDepthStencil::Type BasePassDepthStencilAccess, FViewVisibleCommandsPerView& ViewCommandsPerView, FGlobalDynamicIndexBuffer& DynamicIndexBuffer, FGlobalDynamicVertexBuffer& DynamicVertexBuffer, FGlobalDynamicReadBuffer& DynamicReadBuffer)
    ```
    * Resize the array of the [VisibleLightInfos](#FVisibleLightInfo) in [SceneRender](#FSceneRender)
    * Get the size of [Scene](#FScene)->[Primitives](#FPrimitiveSceneInfo) and get current real time from [ViewFamily](#FSceneViewFamily)
    * [UpdateReflectionSceneData(...)](#UpdateReflectionSceneData(...))
    * Iterate each [Views](#FViewInfo) in [SceneRender](#FSceneRender)
        * Init PrimitiveVisibilityMap, StaticMeshVisibilityMap, etc
        * Init the array of the [VisibleLightInfos](#FVisibleLightInfo)
        * Resize [View](#FViewinfo)::[PrimitiveViewRelevanceMap](#FPrimitiveViewRelevance) by [Scene](#FScene)->[Primitives](#FPrimitiveSceneInfo).Num()
        * [FSceneViewState::GetPrecomputedVisibilityData(...)](#FSceneViewState::GetPrecomputedVisibilityData(...))
        * [ViewState](#FSceneViewState) check [View](#FViewInfo) is frozen
        * Frustum culling
            * Get [HLODTree](#FLODSceneTree) from [Scene](#FScene)->[SceneLODHierarchy](#FLODSceneTree)
            * If [HLODTree](#FLODSceneTree) is Active [UpdateVisibilityStates(...)](#FLODSceneTree::UpdateVisibilityStates(...)) else [ClearVisibilityState(...)](#FLODSceneTree::ClearVisibilityState(...))
            * Execute frustum cull in [FrustumCull(...)](#FrustumCull(...)) by [ParallelFor](#ParallelFor(...)), and store those visibility bit in [Views](#FViewInfo).PotentiallyFadingPrimitiveMap and [Views](#FViewInfo).PrimitiveVisibilityMap
            * Return the number "NumCulledPrimitives" which had been culled, STAT will use it
            * Updated primitive fading states in [UpdatePrimitiveFading(...)](#UpdatePrimitiveFading(...)), like: update UniformBuffer, use uniform to determine which texture will be used in shader, maybe?
        * Set Visibility to "false" in [View](#FViewInfo).PrimitiveVisibilityMap if [View](#FViewInfo).HiddenPrimitives contain it
        * If the view has any primitive is flaged by show only, store them into ShowOnlyPrimitives, hide everything else
        * [View](#FViewInfo).bStaticSceneOnly branch, reflection captures use this, if there's not Proxy->HasStaticLighting() then set visibility to "false"
        * Wireframe branch, cull small objects in wireframe in ortho views, this is important for performance in the editor because wireframe disables any kind of occlusion culling
        * [OcclusionCull(...)](#OcclusionCull(...))
        * Execute [ConditionalUpdateStaticMeshes(...)](#FPrimitiveSceneInfo::ConditionalUpdateStaticMeshes(...)) if any element of [Scene](#FScene).PrimitivesNeedingStaticMeshUpdate is contained by [View](#FViewInfo).PrimitiveVisibilityMap
        * [ComputeAndMarkRelevanceForViewParallel(...)](#ComputeAndMarkRelevanceForViewParallel(...))
        * [GatherDynamicMeshElements(...)](#FSceneRenderer::GatherDynamicMeshElements(...))
        * [SetupMeshPass(...)](#FSceneRenderer::SetupMeshPass(...))
* ### FSceneRenderer::SetupMeshPass(...)
    ```c++
    void FSceneRenderer::SetupMeshPass(FViewInfo& View, FExclusiveDepthStencil::Type BasePassDepthStencilAccess, FViewCommands& ViewCommands)
    ```
    * Setup draw call command in [FParallelMeshDrawCommandPass](#FParallelMeshDrawCommandPass)::[DispatchPassSetup(...)](FParallelMeshDrawCommandPass::DispatchPassSetup(...)), there's only dynamic draw call will be created
* ### FParallelMeshDrawCommandPass::DispatchPassSetup(...)
    ```c++
    void FParallelMeshDrawCommandPass::DispatchPassSetup(
	FScene* Scene,
	const FViewInfo& View,
	EMeshPass::Type PassType,
	FExclusiveDepthStencil::Type BasePassDepthStencilAccess,
	FMeshPassProcessor* MeshPassProcessor,
	const TArray<FMeshBatchAndRelevance, SceneRenderingAllocator>& DynamicMeshElements,
	const TArray<FMeshPassMask, SceneRenderingAllocator>* DynamicMeshElementsPassRelevance,
	int32 NumDynamicMeshElements,
	TArray<const FStaticMeshBatch*, SceneRenderingAllocator>& InOutDynamicMeshCommandBuildRequests,
	int32 NumDynamicMeshCommandBuildRequestElements,
	FMeshCommandOneFrameArray& InOutMeshDrawCommands,
	FMeshPassProcessor* MobileBasePassCSMMeshPassProcessor,
	FMeshCommandOneFrameArray* InOutMobileBasePassCSMMeshDrawCommands
    )
    ```
    * ...TODO
* ### FSceneRenderer::GatherDynamicMeshElements(...)
    ```c++
    void FSceneRenderer::GatherDynamicMeshElements(
	TArray<FViewInfo>& InViews, 
	const FScene* InScene, 
	const FSceneViewFamily& InViewFamily, 
	FGlobalDynamicIndexBuffer& DynamicIndexBuffer,
	FGlobalDynamicVertexBuffer& DynamicVertexBuffer,
	FGlobalDynamicReadBuffer& DynamicReadBuffer,
	const FPrimitiveViewMasks& HasDynamicMeshElementsMasks, 
	const FPrimitiveViewMasks& HasDynamicEditorMeshElementsMasks, 
	const FPrimitiveViewMasks& HasViewCustomDataMasks,
	FMeshElementCollector& Collector)
    ```
    * [Collector](#FMeshElementCollector) and [View](#FViewInfo) are correspondence for each of them
    * Collector Buffers and statistics the number of elements in different [pass](#EMeshPass::Type)
    * ...TODO
* ### ComputeAndMarkRelevanceForViewParallel(...)
    ```c++
    static void ComputeAndMarkRelevanceForViewParallel(
	FRHICommandListImmediate& RHICmdList,
	const FScene* Scene,
	FViewInfo& View,
	FViewCommands& ViewCommands,
	uint8 ViewBit,
	FPrimitiveViewMasks& OutHasDynamicMeshElementsMasks,
	FPrimitiveViewMasks& OutHasDynamicEditorMeshElementsMasks,
	FPrimitiveViewMasks& HasViewCustomDataMasks
	)
    ```
    * Create [FRelevancePackets](#FRelevancePacket) for parallel compute, every Packet have 128 elements, that means if we have 256 primitives will pack into two Packets to execute twice, parallelly
    * Compare them in [ParallelFor](#ParallelFor(...)), every task execute by [AnyThreadTask()](#FRelevancePacket::AnyThreadTask())
    * Collect all [Packets](#FRelevancePacket) and reserve the size of [ViewCommands](#FViewCommands)
    * ForEach [Packet](#FRelevancePacket).[RenderThreadFinalize()](#FRelevancePacket::RenderThreadFinalize()), then don't forget destructing them
    * Set StaticMeshVisibilityMap, StaticMeshFadeOutDitheredLODMap, StaticMeshFadeInDitheredLODMap by MarkMasks which is output of the [FRelevancePacket](#FRelevancePacket)'s [process](#FRelevancePacket::AnyThreadTask())
* ### FRelevancePacket::RenderThreadFinalize()
    ```c++
    void RenderThreadFinalize()
    ```
    * Copy render data to [View](#FViewInfo) and [ViewCommands](#FViewCommands)
    * Prepare translucent self shadow uniform buffers if TranslucentSelfShadowUniformBufferMap not empty (it's a little weird)
* ### FRelevancePacket::AnyThreadTask()
    ```c++
    void AnyThreadTask()
    ```
    * Execute [ComputeRelevance()](#FRelevancePacket::ComputeRelevance()) and [MarkRelevant()](#FRelevancePacket::MarkRelevant())
* ### FRelevancePacket::ComputeRelevance()
    ```c++
    void FRelevancePacket::ComputeRelevance()
    ```
    * Iterate all of the primitives in Input variable which is inited while the begining of the [ComputeAndMarkRelevanceForViewParallel(...)](#ComputeAndMarkRelevanceForViewParallel(...)) execute
        * Get [PrimitiveViewRelevance](#FPrimitiveViewRelevance) via [FPrimitiveSceneInfo](#FPrimitiveSceneInfo) in [FScene](#FScene)
        * Classify [PrimitiveViewRelevances](#FPrimitiveViewRelevance) into different Container like  [RelevantStaticPrimitives](#FRelevancePrimSet) (bit index relate to [Scene](#FScene).[Primitives](#FPrimitiveSceneInfo)), TArray< FMeshDecalBatch> MeshDecalBatches, and [TranslucentPrimCount](#FTranslucenyPrimCount)
* ### FRelevancePacket::MarkRelevant()
    ```c++
    void FRelevancePacket::MarkRelevant()
    ```
    * Iterate RelevantStaticPrimitives, [determine](#ComputeLODForMeshes(...)) which LOD should be use, 
        > **Caution:** static mesh doesn't support mesh render command cache if enable [StaticMeshRelevance](#FStaticMeshBatchRelevance).bDitheredLODTransition, "Don't cache if it requires per view per mesh state for LOD dithering or distance cull fade."
    * Classify primitives into VisibleCachedDrawCommands and DynamicBuildRequests by [PassType](#EMeshPass::Type), if render command has been cached, add that command to VisibleCachedDrawCommands, if not add [StaticMesh](#FStaticMeshBatch) to DynamicBuildRequests
        ```c++
        typedef TArray<FVisibleMeshDrawCommand, TInlineAllocator<AverageMeshBatchNumPerRelevancePacket>> FPassDrawCommandArray;
        typedef TArray<const FStaticMeshBatch*, TInlineAllocator<AverageMeshBatchNumPerRelevancePacket>> FPassDrawCommandBuildRequestArray;
        FPassDrawCommandArray VisibleCachedDrawCommands[EMeshPass::Num];
	    FPassDrawCommandBuildRequestArray DynamicBuildRequests[EMeshPass::Num];
        ```
* ### ComputeLODForMeshes(...)
    ```c++
    FLODMask ComputeLODForMeshes(const TArray<class FStaticMeshBatchRelevance>& StaticMeshRelevances, const FSceneView& View, const FVector4& Origin, float SphereRadius, int32 ForcedLODLevel, float& OutScreenRadiusSquared, float ScreenSizeScale, bool bDitheredLODTransition)
    ```
    * ...TODO
* ### OcclusionCull(...)
    ```c++
    static int32 OcclusionCull(FRHICommandListImmediate& RHICmdList, const FScene* Scene, FViewInfo& View, FGlobalDynamicVertexBuffer& DynamicVertexBuffer)
    ```
    * Cull occluded primitives in the view
    * ...TODO
* ### UpdatePrimitiveFading(...)
    ```c++
    static void UpdatePrimitiveFading(const FScene* Scene, FViewInfo& View)
    ```
    * ...TODO
* ### ParallelFor(...)
    ```c++
    inline void ParallelFor(int32 Num, TFunctionRef<void(int32)> Body, bool bForceSingleThread = false)
    ```
    * ...TODO
* ### FrustumCull(...)
    ```c++
    template<bool UseCustomCulling, bool bAlsoUseSphereTest>
    static int32 FrustumCull(const FScene* Scene, FViewInfo& View)
    ```
    * Parallel execute
        * If distance greater than max draw distance cull it
        * If execute custom culling, [View](#FViewInfo).[CustomVisibilityQuery](#ICustomVisibilityQuery)->IsVisiable    (...)
        * Frustum cull algrithms: [IntersectSphere(...)](#FConvexVolume::IntersectSphere(...)), [IntersectBox(...)](#FConvexVolume::IntersectBox(...))
        * Set visiable primitive bit is **true** that bits stored in [View](#FViewInfo).PrimitiveVisibilityMap and PotentiallyFadingPrimitiveMap
* ### FConvexVolume::IntersectSphere(...)
    ```c++
    bool FConvexVolume::IntersectSphere(const FVector& Origin,const float& Radius) const
    ```
    * ...TODO
* ### FConvexVolume::IntersectBox(...)
    ```c++
    bool FConvexVolume::IntersectBox(const FVector& Origin,const FVector& Extent) const
    ```
    * ...TODO
* ### FLODSceneTree::UpdateVisibilityStates(...)
    ```c++
    void FLODSceneTree::UpdateVisibilityStates(FViewInfo& View)
    ```
    * ...TODO
* ### FLODSceneTree::ClearVisibilityState(...)
    ```c++
    void FLODSceneTree::ClearVisibilityState(FViewInfo& View)
    ```
    * ...TODO
* ### FSceneViewState::GetPrecomputedVisibilityData(...)
    ```c++
    /** 
     * Returns an array of visibility data for the given view position, or NULL if none exists. 
     * The data bits are indexed by VisibilityId of each primitive in the scene.
     * This method decompresses data if necessary and caches it based on the bucket and chunk index in the view state.
    */
    const uint8* FSceneViewState::GetPrecomputedVisibilityData(FViewInfo& View, const FScene* Scene)
    ```
    * If [Scene](#FScene)->PrecomputedVisibilityHandler not null, but it always null
* ### UpdateReflectionSceneData(...)
    ```c++
    void UpdateReflectionSceneData(FScene* Scene)
    ```
    * ...TODO
* ### FDeferredShadingSceneRenderer::PreVisibilityFrameSetup(...)

* ### FSceneRender::PreVisibilityFrameSetup(...)
    * Shrink the [CachedDrawLists](#FCachedPassMeshDrawList)[0/1].[MeshDrawCommands](#FMeshDrawCommand) if [Scene](#FScene)->[Primitives](#FPrimitiveSceneInfo) is empty.
        ```c++
        FCachedPassMeshDrawList CachedDrawLists[EMeshPass::Num];
        namespace EMeshPass
        {
	        enum Type
	        {
	        	DepthPass,
	        	BasePass,
	        	CSMShadowDepth,
	        	Distortion,
	        	Velocity,
	        	TranslucencyStandard,
	        	TranslucencyAfterDOF,
	        	TranslucencyAll, /** Drawing all translucency, regardless of separate or standard.  Used when drawing translucency outside of the main renderer,     eg FRendererModule::DrawTile. */
	        	LightmapDensity,
	        	DebugViewMode, /** Any of EDebugViewShaderMode */
	        	CustomDepth,
	        	MobileBasePassCSM,  /** Mobile base pass with CSM shading enabled */
	        	MobileInverseOpacity,  /** Mobile specific scene capture, Non-cached */
               #if WITH_EDITOR
	        	HitProxy,
	        	HitProxyOpaqueOnly,
	        	EditorSelection,
               #endif

	    	    Num,
	    	    NumBits = 5,
	        };
        }
        ```
    * Update the TArray of the [Primitive](#FPrimitiveSceneInfo), [UpdateStaticMeshes(...)](#FPrimitiveSceneInfo::UpdateStaticMeshes(...))
* ### FPrimitiveSceneInfo::ConditionalUpdateStaticMeshes(...)
    ```c++
    FORCEINLINE void ConditionalUpdateStaticMeshes(FRHICommandListImmediate& RHICmdList)
    {
    		if (NeedsUpdateStaticMeshes())
    		{
    			UpdateStaticMeshes(RHICmdList);
    		}
    }
    ```
    * Execute [UpdateStaticMeshes(...)](#FPrimitiveSceneInfo::UpdateStaticMeshes(...)) if [FPrimitiveSceneInfo](#FPrimitiveSceneInfo)::bNeedsStaticMeshUpdate is 'true'
* ### FPrimitiveSceneInfo::UpdateStaticMeshes(...)
    ```c++
    void FPrimitiveSceneInfo::UpdateStaticMeshes(FRHICommandListImmediate& RHICmdList, bool bReAddToDrawLists);
    ```
    * Add [Primitives](#FPrimitiveSceneInfo) to [SceneRender](#FSceneRender)->PrimitivesNeedingStaticMeshUpdate and [SceneRender](#FSceneRender)->PrimitivesNeedingStaticMeshUpdateWithoutVisibilityCheck if the static mesh is needed update.

        >Think that PrimitivesNeedingStaticMeshUpdate and PrimitivesNeedingStaticMeshUpdateWithoutVisibilityCheck is changeble mesh like desconstruct system Chaos
    * [RemoveCachedMeshDrawCommands()](#RemoveCachedMeshDrawCommands()) as the name suggests
    * [CacheMeshDrawCommands(...)](#CacheMeshDrawCommands(...))
* ### RemoveCachedMeshDrawCommands()
    ```c++
    void FPrimitiveSceneInfo::RemoveCachedMeshDrawCommands()
    ```
    * ...TODO
* ### CacheMeshDrawCommands(...)
    ```c++
    void FPrimitiveSceneInfo::CacheMeshDrawCommands(FRHICommandListImmediate& RHICmdList)
    ```
    * ...TODO