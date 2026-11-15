# HiTechCloud eKYC iOS SDK

> [English](README.md) | **Tiếng Việt**

---

## Tính năng

- Camera Preview
- Liveness
- Nhận diện mặt trước, mặt sau của 4 loại (Chứng minh nhân dân 9 số, Chứng minh nhân dân 12 số, Căn cước công dân, Căn cước công dân gắn chip, Căn cước điện tử, QR VNeID)
- Phát hiện và tự động chụp
- Align ảnh

## Tích hợp SDK

### Yêu cầu

- iOS phiên bản 12.1 trở lên
- Swift phiên bản 5.0 trở lên

### Thêm thư viện kèm SDK

Thêm các thư viện sau vào dự án:

- GoogleMLKit/FaceDetection (phiên bản từ 2.1.0)
- TensorFlowLiteSwift (phiên bản từ 2.5.0)
- OpenCV2 (phiên bản từ 4.3.0)

Có thể manual add các file thư viện đi kèm *.framework vào project hoặc có thể sử dụng CocoaPods:

```
- pod 'GoogleMLKit/FaceDetection', '~> 2.1.0'
- pod 'TensorFlowLiteSwift', '~> 2.5.0'
- pod 'OpenCV2', '~> 4.3.0'
```

### Thêm SDK

(Project -> Target -> General)

- Thêm file HITECHCLOUD_EKYC.framework, và set Embed & Sign

### Cách sử dụng

- Import HITECHCLOUD_EKYC trong dự án
- Import AVFoundation trong dự án
- Xin quyền Privacy - Camera Usage Description ở Info.plist
- Kế thừa HITECHCLOUDEKYCCameraDelegate, HITECHCLOUDEKYCIdCardDetectorDelegate, HITECHCLOUDEKYCFaceDetectorDelegate
- Khởi tạo view HITECHCLOUDEKYC và cài đặt các thuộc tính

#### Nhận diện khuôn mặt

```
import UIKit
import HITECHCLOUD_EKYC
import AVFoundation
```

```
class FaceDetectorVC: UIViewController {

  @IBOutlet weak var cameraView: UIView!
  @IBOutlet weak var correctView: UIImageView!

  var camera: HITECHCLOUDEKYC?

  override func viewDidLoad() {
    super.viewDidLoad()
    camera = HITECHCLOUDEKYC.init(frameCameraPreview: self.cameraView.bounds, frameCorrect: self.correctView.frame)
    if let camera = self.camera {
      self.cameraView.addSubview(camera.cameraPreview)
    }
    self.cameraView.bringSubviewToFront(self.correctView)
    self.cameraView.bringSubviewToFront(self.imageCropView)

    // Set Delegate
    camera?.cameraDelegate = self
    camera?.faceDetectorDelegate = self

    //Set Type
    camera?.isType = .face

    //Show debug view
    camera?.showDebug = true

    //Position
    camera?.cameraPosition = .front

    // Start camera session
    camera?.startSession()
  }

  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    // Restart session - restart the camera if it was previously paused
    self.camera?.restartSession()
  }

  override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    // Pause camera session - pause the camera if needed
    self.camera?.stopSession()
  }
}
```

Dữ liệu sẽ được xử lý và trả kết quả thông qua các phương thức sau:

```
extension FaceDetectorVC: HITECHCLOUDEKYCCameraDelegate {

  /// User denied camera permission
  func cameraPermissionCameraDenied(status: AVAuthorizationStatus) {
  }

  // Camera started successfully
  func cameraDidStart() {
  }

  /// Camera session failed to start
  func cameraDidFail(error: LocalizedError) {
  }

  /// Camera position changed
  func cameraChangePosition(newPosision: AVCaptureDevice.Position) {
  }
}
```

```
extension FaceDetectorVC: HITECHCLOUDEKYCFaceDetectorDelegate {
  /// Result: valid reports whether a face was detected and
  /// the conditions (looking straight ahead, no head tilt) are met
  func faceDetectionResult(valid: Bool, image: UIImage?) {
    // This method is still called every time a face is received
    // Set a flag when an image is received to stop running this block as needed
    DispatchQueue.main.async {
      self.correctView.image = self.correctView.image?.withRenderingMode(.alwaysTemplate)
      self.correctView.tintColor = valid ? UIColor.green : UIColor.red
    }

    if let imageResult = image, valid {
      // Image received
      // flag
    }
  }
}
```

#### Nhận diện ID Card

```
import UIKit
import HITECHCLOUD_EKYC
import AVFoundation
class IdCardDetectorVC: UIViewController {

  @IBOutlet weak var cameraView: UIView!
  @IBOutlet weak var correctView: UIImageView!

  var camera: HITECHCLOUDEKYC?

  override func viewDidLoad() {
    super.viewDidLoad()
    camera = HITECHCLOUDEKYC.init(frameCameraPreview: self.cameraView.bounds, frameCorrect: self.correctView.frame)
    if let camera = self.camera {
      self.cameraView.addSubview(camera.cameraPreview)
    }

    self.cameraView.bringSubviewToFront(self.correctView)

    // Set Delegate
    camera?.cameraDelegate = self
    camera?.idCardDetectorDelegate = self

    // Set Type
    camera?.isType = .idCard

    // Show debug view
    camera?.showDebug = true

    // Position
    camera?.cameraPosition = .back

    // Start camera session
    camera?.startSession()
  }

  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    // Restart session - restart the camera if it was previously paused
    self.camera?.restartSession()
  }

  override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    // Pause camera session - pause the camera if needed
    self.camera?.stopSession()
  }
}
```

Dữ liệu sẽ được xử lý và trả kết quả thông qua các phương thức sau:

```
extension IdCardDetectorVC: HITECHCLOUDEKYCIdCardDetectorDelegate {
  func idCardDetectionResult(valid: Bool, result: IdCardResult?) {
    // This method is still called every time an ID card is received
    // Set a flag when an image is received to stop running this block as needed
    DispatchQueue.main.async {
      self.correctView.image = self.correctView.image?.withRenderingMode(.alwaysTemplate)
      self.correctView.tintColor = valid ? UIColor.green : UIColor.red
    }
    if let result = result, valid {
      let type: IdCardType = result.type
      let image: UIImage = result.image
      // Data received
      //flag
    }
  }
}
```
