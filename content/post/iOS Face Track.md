---
title: "iOS Face Track"
date: 2021-04-20T16:17:52+08:00
tags : [ "iOS", "Face Track"]
layout: post
type:  "post"
draft: false
---

## How does it work?

- Using `VNDetectFaceLandmarksRequestRevision3`
```objc
Starting with iOS 13, you will get a different set of points (VNDetectFaceLandmarksRequestRevision3) 
```

- Get size of camera layer.
```objc
AVCaptureVideoDataOutput *output = [[[self.videoCamera captureSession] outputs] lastObject];
NSDictionary* outputSettings = [output videoSettings];

long width  = [[outputSettings objectForKey:@"Width"]  longValue];
long height = [[outputSettings objectForKey:@"Height"] longValue];
```

- Conver points

- 1. Method one
```objc
size_t width = CVPixelBufferGetWidth(CVPixelBufferRef);
size_t height = CVPixelBufferGetHeight(CVPixelBufferRef);

CGSize size = CGSizeMake(width, height);

CGFloat scaleX = self.filterView.layer.frame.size.width / size.width;
CGFloat scaleY = self.filterView.layer.frame.size.height / size.height;

CGAffineTransform transform = CGAffineTransformTranslate(CGAffineTransformMakeScale(scaleX, -scaleY), 0, -1);

CGRect faceBoundingBoxOnScreen = VNImageRectForNormalizedRect(CGRectApplyAffineTransform(observedFace.boundingBox, transform), size.width, size.height);
```

- 2. Method two
```objc
CGRect faceBoundingBoxOnScreen = CGRectZero;
faceBoundingBoxOnScreen.size.height = self.filterView.layer.frame.size.height * observedFace.boundingBox.size.height;
faceBoundingBoxOnScreen.size.width = self.filterView.layer.frame.size.width * observedFace.boundingBox.size.width;
faceBoundingBoxOnScreen.origin.x = observedFace.boundingBox.origin.x * self.filterView.layer.frame.size.width;
faceBoundingBoxOnScreen.origin.y = observedFace.boundingBox.origin.y * self.filterView.layer.frame.size.height;
```

- Eyes

- 1. Method one
```objc
for (int i = 0; i < eye.pointCount; i++) {
    CGPoint eyePoint = eye.normalizedPoints[i];
    CGRect faceBounds = VNImageRectForNormalizedRect(screenBoundingBox, size.width, size.height);

    CGAffineTransform transform = CGAffineTransformScale(CGAffineTransformMakeTranslation(faceBounds.origin.x, faceBounds.origin.y), faceBounds.size.width, faceBounds.size.height);

    eyePoint = CGPointApplyAffineTransform(eyePoint, transform);

    CGFloat scaleX = self.filterView.layer.frame.size.width / size.width;
    CGFloat scaleY = self.filterView.layer.frame.size.height / size.height;

    transform = CGAffineTransformTranslate(CGAffineTransformMakeScale(scaleX, -scaleY), 0, -size.height);

    eyePoint = CGPointApplyAffineTransform(eyePoint, transform);
}
```

- 2. Method two
```objc
const CGPoint *pointsInImage = [eye pointsInImageOfSize:CGSizeMake(size.width, size.height)];
for (int i = 0; i < eye.pointCount; i++) {
    CGPoint eyePoint = pointsInImage[i];

    CGFloat scaleX = self.filterView.layer.frame.size.width / size.width;
    CGFloat scaleY = self.filterView.layer.frame.size.height / size.height;
    
    CGAffineTransform transform = CGAffineTransformTranslate(CGAffineTransformMakeScale(scaleX, -scaleY), 0, -size.height);

    eyePoint = CGPointApplyAffineTransform(eyePoint, transform);

    newEyePoints[i] = eyePoint;
    [newEyePointsArray addObject:[NSValue valueWithCGPoint:eyePoint]];
}
```

- All points
```objc
NSMutableArray *newAllPointsArray = [NSMutableArray array];
const CGPoint *pointsInImage = [landmarks.allPoints pointsInImageOfSize:CGSizeMake(size.width, size.height)];
for (int i = 0; i < landmarks.allPoints.pointCount; i++) {
    CGPoint eyePoint = pointsInImage[i];

    CGFloat scaleX = (self.filterView.layer.frame.size.width / size.width) * (size.height / self.filterView.layer.frame.size.width);
    CGFloat scaleY = (self.filterView.layer.frame.size.height / size.height) * (size.width / self.filterView.layer.frame.size.height);
    
    CGAffineTransform transform = CGAffineTransformTranslate(CGAffineTransformMakeScale(scaleX, -scaleY), 0, -size.height);
    
    eyePoint = CGPointApplyAffineTransform(eyePoint, transform);
    
    [newAllPointsArray addObject:[NSValue valueWithCGPoint:eyePoint]];
}
```

