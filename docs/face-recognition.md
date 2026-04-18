# Face Recognition Implementation Plan (AWS Rekognition)

## Overview

We will use **AWS Rekognition** `CompareFaces` API for verifying user identity (1:1 matching). This solution is robust, industry-standard, and cost-effective.

## 1. Pricing Structure (Pay-As-You-Go)

AWS Rekognition has no upfront cost or minimum fee.

- **Free Tier**: First 12 months = **1,000 face comparisons/month** for free.
- **Standard Pricing**: **$0.001 per image**.
    - 1,000 verifications = $1.00
    - 10,000 verifications = $10.00

## 2. Architecture

**Security Critical**: The AWS SDK must NOT be installed in the React Native app to avoid exposing secret keys.

- **Frontend (Mobile)**: Captures two images (e.g., ID card & Selfie) using `expo-camera` or `expo-image-picker`.
- **Backend (Supabase Edge Function)**: Holds the AWS credentials securely. It receives the images, calls AWS, and returns the result.

## 3. Implementation Steps

### A. AWS Configuration

1.  **Create IAM User**: Create a programatic user in AWS Console.
2.  **Permissions**: Attach the `AmazonRekognitionFullAccess` policy (or a custom policy allowing just `rekognition:CompareFaces`).
3.  **Credentials**: Generate and save the `Access Key ID` and `Secret Access Key`.
4.  **Supabase Secrets**: Add these keys to your Edge Function environment variables.

### B. Backend (Supabase Edge Function)

**Install SDK**:
`npm install @aws-sdk/client-rekognition`

**Function Code (Draft)**:

```typescript
import {
    RekognitionClient,
    CompareFacesCommand,
} from "@aws-sdk/client-rekognition";

const client = new RekognitionClient({
    region: "us-east-1", // or your preferred region
    credentials: {
        accessKeyId: Deno.env.get("AWS_ACCESS_KEY_ID")!,
        secretAccessKey: Deno.env.get("AWS_SECRET_ACCESS_KEY")!,
    },
});

export const compareFaces = async (
    sourceImageBytes: Uint8Array,
    targetImageBytes: Uint8Array,
) => {
    const command = new CompareFacesCommand({
        SourceImage: { Bytes: sourceImageBytes }, // e.g. Profile Photo
        TargetImage: { Bytes: targetImageBytes }, // e.g. Selfie
        SimilarityThreshold: 80, // Minimum confidence (0-100)
    });

    try {
        const response = await client.send(command);
        const bestMatch = response.FaceMatches?.[0];

        return {
            matched: !!bestMatch,
            confidence: bestMatch ? bestMatch.Similarity : 0,
        };
    } catch (error) {
        console.error("Rekognition Error:", error);
        throw new Error("Face verification failed");
    }
};
```

### C. Frontend (React Native)

1.  Capture image as `base64`.
2.  Send POST request to your Supabase Edge Function with both images.
3.  Handle the `matched` boolean response.

## Next Action Items

- [ ] Set up AWS Account & IAM User.
- [ ] Create `verify-user` Edge Function.
- [ ] Connect Frontend `VerifyPhoto.tsx` to the Edge Function.
