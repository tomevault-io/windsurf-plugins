---
trigger: always_on
description: Run Edge AI and On-Device Machine Learning Standards Check
---

# IoT: Edge AI & On-Device Machine Learning Standards

<audit_rules>
- You MUST implement proper model optimization for edge devices with quantization and pruning.
- You MUST ensure proper on-device inference with memory and compute constraints.
- You MUST implement proper federated learning with privacy-preserving aggregation.
- You MUST configure proper model update mechanisms with delta updates and compression.
- You MUST ensure proper edge-to-cloud coordination with fallback mechanisms.
- You MUST implement proper on-device data preprocessing and feature extraction.
- You MUST configure proper model monitoring and performance optimization on edge devices.
- You MUST ensure proper edge AI security with model protection and secure inference.
- You MUST implement proper edge AI lifecycle management and version control.
- You MUST ensure proper resource management and power optimization for edge AI.
</audit_rules>

<example_good>
```typescript
// Edge AI Implementation

export class EdgeAIManager {
  constructor(
    private modelOptimizer: EdgeModelOptimizer,
    private inferenceEngine: EdgeInferenceEngine,
    private federatedLearning: FederatedLearningEngine,
    private modelUpdater: EdgeModelUpdater,
    private resourceManager: EdgeResourceManager,
    private securityManager: EdgeSecurityManager
  ) {}

  async deployEdgeModel(deployment: EdgeModelDeployment): Promise<EdgeModelDeploymentResult> {
    // Validate deployment requirements
    const validation = await this.validateDeployment(deployment);
    if (!validation.valid) {
      throw new Error(`Invalid deployment: ${validation.errors.join(', ')}`);
    }

    // Optimize model for edge device
    const optimizedModel = await this.modelOptimizer.optimizeForEdge(deployment.model, {
      targetDevice: deployment.deviceId,
      constraints: deployment.constraints,
      optimizationLevel: deployment.optimizationLevel || 'high',
    });

    // Validate optimized model
    const modelValidation = await this.validateOptimizedModel(optimizedModel, deployment);
    if (!modelValidation.valid) {
      throw new Error(`Model validation failed: ${modelValidation.errors.join(', ')}`);
    }

    // Deploy model to edge device
    const deploymentResult = await this.deployToDevice(deployment.deviceId, optimizedModel);

    // Setup inference engine
    const inferenceSetup = await this.inferenceEngine.setup({
      deviceId: deployment.deviceId,
      model: optimizedModel,
      configuration: deployment.inferenceConfig,
    });

    // Configure monitoring
    const monitoringConfig = await this.setupModelMonitoring(deployment.deviceId, optimizedModel);

    return {
      deploymentId: deploymentResult.id,
      deviceId: deployment.deviceId,
      modelId: optimizedModel.id,
      modelSize: optimizedModel.size,
      inferenceSetup,
      monitoringConfig,
      deploymentTimestamp: Date.now(),
    };
  }

  async runInference(request: EdgeInferenceRequest): Promise<EdgeInferenceResult> {
    // Validate inference request
    const validation = await this.inferenceEngine.validateRequest(request);
    if (!validation.valid) {
      throw new Error(`Invalid inference request: ${validation.errors.join(', ')}`);
    }

    // Check device resources
    const resourceCheck = await this.resourceManager.checkResources(request.deviceId);
    if (!resourceCheck.sufficient) {
      throw new Error(`Insufficient resources: ${resourceCheck.issues.join(', ')}`);
    }

    // Preprocess input data
    const preprocessedData = await this.preprocessInput(request.input, request.preprocessing);

    // Run inference
    const inferenceResult = await this.inferenceEngine.infer({
      deviceId: request.deviceId,
      input: preprocessedData,
      modelId: request.modelId,
      options: request.options,
    });

    // Postprocess results
    const postprocessedResult = await this.postprocessOutput(
      inferenceResult.output,
      request.postprocessing
    );

    // Log inference metrics
    await this.logInferenceMetrics(request, inferenceResult);

    return {
      ...inferenceResult,
      output: postprocessedResult,
      deviceId: request.deviceId,
      inferenceTime: inferenceResult.inferenceTime,
      memoryUsage: inferenceResult.memoryUsage,
      timestamp: Date.now(),
    };
  }

  async participateInFederatedLearning(participation: FederatedLearningParticipation): Promise<FederatedLearningResult> {
    // Validate participation request
    const validation = await this.validateFederatedLearningParticipation(participation);
    if (!validation.valid) {
      throw new Error(`Invalid federated learning participation: ${validation.errors.join(', ')}`);
    }

    // Get local training data
    const localData = await this.getLocalTrainingData(participation.deviceId, participation.dataConfig);

    // Train local model
    const localTraining = await this.trainLocalModel(participation, localData);

    // Apply differential privacy
    const privateModel = await this.applyDifferentialPrivacy(localTraining.model, participation.privacyConfig);

    // Submit to federated learning coordinator
    const submission = await this.federatedLearning.submitUpdate({
      deviceId: participation.deviceId,
      modelUpdate: privateModel,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TrevorPLam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
