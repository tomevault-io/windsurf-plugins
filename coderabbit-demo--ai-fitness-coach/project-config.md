---
trigger: always_on
description: - **Use Consistent AI Client Setup:**
---

# AI Integration Rules for AI Fitness Coach

## **Vercel AI SDK Patterns**

- **Use Consistent AI Client Setup:**
  ```typescript
  // ✅ DO: Configure AI client with proper error handling
  import { openai } from '@ai-sdk/openai'
  import { generateText, generateObject } from 'ai'
  
  const aiClient = openai({
    apiKey: process.env.OPENAI_API_KEY,
    // Configure for health data processing
    maxRetries: 3,
    timeout: 30000
  })
  ```

- **Structure Health Data Prompts:**
  ```typescript
  // ✅ DO: Use structured prompts for health analysis
  const generateFitnessRecommendation = async (userData: UserHealthData) => {
    const systemPrompt = `You are an AI fitness coach focused on user success and incremental improvements. 
    Analyze the user's data and provide personalized, actionable recommendations.
    
    User Context:
    - Goals: ${userData.goals}
    - Current metrics: Weight ${userData.currentWeight}kg, Activity level: ${userData.activityLevel}
    - Recent patterns: ${userData.recentPatterns}
    
    Provide recommendations that are:
    1. Specific and actionable
    2. Realistic given their current situation
    3. Focus on small, incremental improvements
    4. Consider their emotional state and external factors`
    
    const { text } = await generateText({
      model: aiClient,
      system: systemPrompt,
      prompt: `Generate personalized fitness recommendations for today.`
    })
    
    return text
  }
  ```

## **Food Recognition & Calorie Estimation**

- **Image Analysis for Food Logging:**
  ```typescript
  // ✅ DO: Multi-step food analysis with validation
  export async function analyzeFoodImage(imageBase64: string): Promise<NutritionEstimate> {
    // Step 1: Initial food identification
    const initialAnalysis = await generateObject({
      model: aiClient,
      schema: z.object({
        foodItems: z.array(z.object({
          name: z.string(),
          estimatedQuantity: z.string(),
          confidence: z.number().min(0).max(1)
        })),
        totalCaloriesEstimate: z.number(),
        confidence: z.number().min(0).max(1)
      }),
      messages: [
        {
          role: 'user',
          content: [
            {
              type: 'text',
              text: 'Analyze this food image and identify all food items with calorie estimates. Be conservative with estimates - it\'s better to slightly underestimate than overestimate.'
            },
            {
              type: 'image',
              image: imageBase64
            }
          ]
        }
      ]
    })
    
    // Step 2: Validation and refinement
    const validation = await generateObject({
      model: aiClient,
      schema: z.object({
        isReasonable: z.boolean(),
        adjustedCalories: z.number(),
        reasoning: z.string()
      }),
      prompt: `Validate this food analysis: ${JSON.stringify(initialAnalysis.object)}
      
      Check if the calorie estimates are reasonable. Adjust if needed.`
    })
    
    return {
      ...initialAnalysis.object,
      finalCalories: validation.object.adjustedCalories,
      validationNotes: validation.object.reasoning
    }
  }

- **Handle AI Uncertainty:**
  ```typescript
  // ✅ DO: Gracefully handle low-confidence results
  export function processNutritionAnalysis(result: NutritionEstimate) {
    if (result.confidence < 0.7) {
      return {
        ...result,
        requiresUserConfirmation: true,
        message: "I'm not entirely sure about this estimate. Please review and adjust if needed."
      }
    }
    
    return {
      ...result,
      requiresUserConfirmation: false,
      message: "Nutrition estimate looks good!"
    }
  }
  ```

## **Holistic Health Analysis**

- **Multi-Factor Recommendation Engine:**
  ```typescript
  // ✅ DO: Comprehensive health data analysis
  export async function generateDailyRecommendations(
    userId: string
  ): Promise<DailyRecommendations> {
    // Gather user data from multiple sources
    const [profile, recentWeights, nutrition, sleep, mood] = await Promise.all([
      getUserProfile(userId),
      getRecentWeightLogs(userId, 7), // Last 7 days
      getRecentNutritionLogs(userId, 3), // Last 3 days
      getRecentSleepData(userId, 3),
      getRecentMoodLogs(userId, 7)
    ])
    
    const contextPrompt = `
    User Profile: ${JSON.stringify(profile)}
    Weight Trend: ${analyzeWeightTrend(recentWeights)}
    Nutrition Pattern: ${analyzeNutritionPattern(nutrition)}
    Sleep Quality: ${analyzeSleepPattern(sleep)}
    Mood Trend: ${analyzeMoodTrend(mood)}
    `
    
    const recommendations = await generateObject({
      model: aiClient,
      schema: DailyRecommendationsSchema,
      system: `You are an expert fitness coach focused on user success through incremental improvements.
      Analyze all available data to provide personalized recommendations.
      
      Key principles:
      - Small, achievable changes
      - Consider emotional state and life circumstances
      - Focus on building sustainable habits
      - Be encouraging and supportive`,
      prompt: `Based on this user's data, generate today's recommendations: ${contextPrompt}`
    })
    
    return recommendations.object
  }
  ```

## **Background AI Processing with Inngest**

- **Queue AI Analysis Jobs:**
  ```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coderabbit-demo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
