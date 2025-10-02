# Media Service Lambda Function Documentation

## Overview

The Media Service is a specialized Lambda function that handles image upload, processing, and optimization for the HarborList platform. It provides secure file upload via presigned URLs, automatic thumbnail generation using Sharp image processing, and multi-format image optimization with S3 integration.

## Architecture

### Function Structure
- **Runtime**: Node.js 18 with TypeScript
- **Handler**: `backend/src/media/index.ts`
- **Dependencies**: AWS SDK v3, Sharp image processing library
- **Storage**: Amazon S3 for original images and thumbnails
- **Processing**: Event-driven image processing via S3 triggers
- **Authentication**: JWT-based user authentication required

### Design Principles
- **Secure Upload**: Presigned URLs for direct S3 upload
- **Automatic Processing**: Event-driven thumbnail generation
- **Multi-Format Support**: JPEG and WebP format generation
- **Performance Optimization**: Multiple thumbnail sizes for responsive design
- **User Isolation**: User-specific folder structure in S3

## API Endpoints

### POST `/media/upload`
**Purpose**: Generate presigned URL for secure image upload

**Authentication**: Required (JWT token)

**Request Headers**:
```
Content-Type: multipart/form-data
Authorization: Bearer <jwt_token>
```

**Response**:
```typescript
{
  uploadId: string; // Unique identifier for the upload
  uploadUrl: string; // Presigned S3 URL for direct upload
  url: string; // Final URL where image will be accessible
  thumbnail: string; // URL for generated thumbnail
  metadata: {
    size: number; // File size in bytes (0 for presigned URL)
    dimensions: {
      width: number; // Image width in pixels
      height: number; // Image height in pixels
    };
    format: string; // Image format (e.g., "JPEG")
  };
}
```

**Example Response**:
```json
{
  "uploadId": "1640995200000-abc123def",
  "uploadUrl": "https://boat-media-bucket.s3.amazonaws.com/user123/1640995200000-abc123def?X-Amz-Algorithm=...",
  "url": "https://boat-media-bucket.s3.amazonaws.com/user123/1640995200000-abc123def",
  "thumbnail": "https://boat-thumbnails-bucket.s3.amazonaws.com/user123/1640995200000-abc123def_thumb.jpg",
  "metadata": {
    "size": 0,
    "dimensions": { "width": 0, "height": 0 },
    "format": "JPEG"
  }
}
```

## Image Processing Pipeline

### Upload Flow
1. **Client Request**: User requests upload URL
2. **Authentication**: Verify JWT token and extract user ID
3. **Generate Presigned URL**: Create secure S3 upload URL
4. **Return URLs**: Provide upload and access URLs to client
5. **Direct Upload**: Client uploads directly to S3 using presigned URL
6. **S3 Event Trigger**: S3 ObjectCreated event triggers processing

### Processing Flow
1. **S3 Event**: ObjectCreated event triggers `processImageHandler`
2. **Image Retrieval**: Download original image from S3
3. **Thumbnail Generation**: Create multiple thumbnail sizes
4. **Format Optimization**: Generate WebP version for modern browsers
5. **Storage**: Save processed images to thumbnails bucket

## Presigned URL Implementation

### URL Generation
```typescript
const uploadUrl = await getSignedUrl(
  s3Client,
  new PutObjectCommand({
    Bucket: MEDIA_BUCKET,
    Key: fileName, // Format: userId/fileId
    ContentType: 'image/jpeg',
  }),
  { expiresIn: 3600 } // 1 hour expiration
);
```

### Security Features
- **Time-Limited**: URLs expire after 1 hour
- **User Isolation**: Files organized by user ID
- **Content-Type Restriction**: Enforces image content types
- **Authentication Required**: JWT token validation before URL generation

### File Organization
```
S3 Bucket Structure:
├── user123/
│   ├── 1640995200000-abc123def.jpg (original)
│   ├── 1640995200000-xyz789ghi.jpg (original)
│   └── ...
├── user456/
│   ├── 1640995300000-def456abc.jpg (original)
│   └── ...
```

## Image Processing with Sharp

### Thumbnail Generation
```typescript
const thumbnailSizes = [
  { width: 150, height: 150, suffix: '_thumb_150' }, // Small thumbnails
  { width: 300, height: 300, suffix: '_thumb_300' }, // Medium thumbnails
  { width: 600, height: 400, suffix: '_thumb_600' }, // Large thumbnails
];

for (const size of thumbnailSizes) {
  const thumbnail = await sharp(imageBuffer)
    .resize(size.width, size.height, {
      fit: 'cover', // Crop to fit dimensions
      position: 'center', // Center the crop
    })
    .jpeg({ quality: 85 }) // High quality JPEG
    .toBuffer();
}
```

### Processing Features
- **Multiple Sizes**: 150x150, 300x300, 600x400 pixel thumbnails
- **Smart Cropping**: Center-focused crop to maintain aspect ratio
- **Quality Optimization**: 85% JPEG quality for size/quality balance
- **Format Conversion**: Automatic WebP generation for modern browsers

### WebP Optimization
```typescript
const webpImage = await sharp(imageBuffer)
  .webp({ quality: 85 })
  .toBuffer();

const webpKey = key.replace(/\.[^/.]+$/, '.webp');
```

**Benefits**:
- **Smaller File Sizes**: 25-35% smaller than JPEG
- **Modern Browser Support**: Excellent quality-to-size ratio
- **Fallback Support**: JPEG versions available for older browsers

## S3 Event Processing

### Event Handler Configuration
```typescript
export const processImageHandler = async (event: any) => {
  for (const record of event.Records) {
    if (record.eventName.startsWith('ObjectCreated')) {
      const bucket = record.s3.bucket.name;
      const key = record.s3.object.key;
      
      try {
        await processImage(bucket, key);
      } catch (error) {
        console.error(`Failed to process image ${key}:`, error);
      }
    }
  }
};
```

### Event Trigger Setup
- **Event Type**: `s3:ObjectCreated:*`
- **Source Bucket**: Media bucket for original images
- **Filter**: Image file extensions (jpg, jpeg, png, gif)
- **Async Processing**: Non-blocking thumbnail generation

### Error Handling
- **Individual Processing**: Each image processed independently
- **Error Isolation**: Failed processing doesn't affect other images
- **Retry Logic**: S3 events automatically retry on Lambda failures
- **Logging**: Detailed error logging for troubleshooting

## Storage Architecture

### S3 Bucket Configuration

#### Media Bucket (Original Images)
```typescript
const MEDIA_BUCKET = process.env.MEDIA_BUCKET; // e.g., "boat-media-bucket"
```
- **Purpose**: Store original uploaded images
- **Access**: Private with presigned URL access
- **Lifecycle**: Configurable retention policies
- **Versioning**: Optional for data protection

#### Thumbnails Bucket (Processed Images)
```typescript
const THUMBNAILS_BUCKET = process.env.THUMBNAILS_BUCKET; // e.g., "boat-thumbnails-bucket"
```
- **Purpose**: Store generated thumbnails and optimized images
- **Access**: Public read access for web serving
- **CDN Integration**: CloudFront distribution for global delivery
- **Cost Optimization**: Intelligent tiering for infrequent access

### File Naming Convention
```typescript
// Original image
userId/fileId.jpg

// Thumbnails
userId/fileId_thumb_150.jpg  // 150x150 thumbnail
userId/fileId_thumb_300.jpg  // 300x300 thumbnail
userId/fileId_thumb_600.jpg  // 600x400 thumbnail

// WebP version
userId/fileId.webp
```

## Image Metadata and Validation

### Content Type Validation
```typescript
const contentType = event.headers['content-type'] || event.headers['Content-Type'];

if (!contentType?.includes('multipart/form-data')) {
  return createErrorResponse(400, 'INVALID_CONTENT_TYPE', 'Content-Type must be multipart/form-data', requestId);
}
```

### Supported Formats
- **Input**: JPEG, PNG, GIF, WebP
- **Output**: JPEG (thumbnails), WebP (optimized)
- **Quality**: 85% for optimal size/quality balance

### File Size Limits
```typescript
// Recommended limits (configured at API Gateway/Lambda level)
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
const MAX_DIMENSION = 4096; // 4K resolution
```

## Stream Processing Utilities

### Buffer Conversion
```typescript
async function streamToBuffer(stream: any): Promise<Buffer> {
  const chunks: Buffer[] = [];
  
  return new Promise((resolve, reject) => {
    stream.on('data', (chunk: Buffer) => chunks.push(chunk));
    stream.on('error', reject);
    stream.on('end', () => resolve(Buffer.concat(chunks)));
  });
}
```

**Features**:
- **Memory Efficient**: Streams large files without loading entirely into memory
- **Error Handling**: Proper stream error propagation
- **Promise-Based**: Modern async/await compatibility

## Error Handling

### Comprehensive Error Responses
```typescript
// Authentication errors
{ error: { code: 'UNAUTHORIZED', message: 'User not authenticated' } }

// Content type errors
{ error: { code: 'INVALID_CONTENT_TYPE', message: 'Content-Type must be multipart/form-data' } }

// Method errors
{ error: { code: 'METHOD_NOT_ALLOWED', message: 'Method GET not allowed' } }

// Processing errors
{ error: { code: 'UPLOAD_ERROR', message: 'Failed to process media upload' } }
```

### Error Handling Strategy
1. **Input Validation**: Verify content type and authentication
2. **S3 Operations**: Handle AWS service errors
3. **Image Processing**: Catch Sharp processing errors
4. **Stream Operations**: Handle stream reading errors
5. **Generic Fallback**: Catch unexpected errors

## Performance Optimization

### Lambda Configuration
```typescript
// Recommended Lambda settings
{
  memorySize: 1024, // MB - Required for Sharp image processing
  timeout: 300, // seconds - 5 minutes for large image processing
  environment: {
    NODE_OPTIONS: '--max-old-space-size=1024'
  }
}
```

### Processing Optimizations
- **Parallel Processing**: Multiple thumbnail sizes generated concurrently
- **Buffer Reuse**: Single image buffer for multiple operations
- **Quality Settings**: Optimized for web delivery
- **Format Selection**: WebP for modern browsers, JPEG fallback

### Memory Management
```typescript
// Sharp memory optimization
sharp.cache(false); // Disable cache for Lambda environment
sharp.concurrency(1); // Single thread processing
```

## Integration Points

### Frontend Integration
```typescript
// React component example
const uploadImage = async (file: File) => {
  // 1. Get presigned URL
  const { uploadUrl, url } = await mediaService.getUploadUrl();
  
  // 2. Upload directly to S3
  await fetch(uploadUrl, {
    method: 'PUT',
    body: file,
    headers: { 'Content-Type': file.type }
  });
  
  // 3. Use returned URL in listing
  return url;
};
```

### Listing Service Integration
```typescript
// Listing creation with images
const listing = {
  // ... other fields
  images: [
    'https://boat-media-bucket.s3.amazonaws.com/user123/image1.jpg',
    'https://boat-media-bucket.s3.amazonaws.com/user123/image2.jpg'
  ],
  thumbnails: [
    'https://boat-thumbnails-bucket.s3.amazonaws.com/user123/image1_thumb_300.jpg',
    'https://boat-thumbnails-bucket.s3.amazonaws.com/user123/image2_thumb_300.jpg'
  ]
};
```

## Security Considerations

### Access Control
- **Authentication**: JWT token required for upload URL generation
- **User Isolation**: Files organized by user ID
- **Presigned URL Security**: Time-limited access (1 hour)
- **Content Validation**: File type and size restrictions

### Data Protection
- **Encryption**: S3 server-side encryption enabled
- **Access Logging**: S3 access logs for audit trail
- **CORS Configuration**: Restricted cross-origin access
- **IAM Policies**: Least privilege access for Lambda functions

## Monitoring and Analytics

### Key Metrics
- **Upload Volume**: Number of images uploaded per time period
- **Processing Time**: Average thumbnail generation time
- **Error Rates**: Failed uploads and processing errors
- **Storage Usage**: S3 bucket size and growth trends
- **CDN Performance**: CloudFront cache hit rates

### CloudWatch Metrics
```typescript
// Custom metrics to track
{
  'MediaService/UploadsCount': uploadCount,
  'MediaService/ProcessingDuration': processingTime,
  'MediaService/ThumbnailsGenerated': thumbnailCount,
  'MediaService/ErrorRate': errorRate
}
```

## Environment Configuration

### Required Environment Variables
```bash
# S3 Configuration
MEDIA_BUCKET=boat-media-bucket
THUMBNAILS_BUCKET=boat-thumbnails-bucket
AWS_REGION=us-east-1

# Processing Configuration
MAX_FILE_SIZE=10485760 # 10MB in bytes
THUMBNAIL_QUALITY=85
WEBP_QUALITY=85
```

### S3 Bucket Policies
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::ACCOUNT:role/MediaServiceRole"},
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::boat-media-bucket/*"
    }
  ]
}
```

## Future Enhancements

### Advanced Processing
```typescript
// Image analysis and metadata extraction
interface ImageAnalysis {
  faces: number;
  objects: string[];
  colors: string[];
  quality: number;
  appropriateness: boolean;
}

// Video processing support
interface VideoProcessing {
  thumbnailExtraction: boolean;
  formatConversion: string[];
  compressionLevels: number[];
}
```

### Performance Improvements
- **CDN Integration**: CloudFront for global image delivery
- **Lazy Loading**: Progressive image loading strategies
- **Caching**: Redis cache for frequently accessed images
- **Batch Processing**: Multiple image processing in single invocation

This Media Service provides robust, scalable image handling with automatic optimization and secure upload capabilities for the HarborList platform.