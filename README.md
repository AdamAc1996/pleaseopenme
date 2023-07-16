import SwiftUI
import CoreLocation

@main
struct JJgameApp: App {
    let persistenceController = PersistenceController.shared

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, persistenceController.container.viewContext)
        }
    }
}

struct ContentView: View {
    @State private var image: UIImage?
    @State private var location: CLLocation?
    @State private var isShowingImagePicker = false
    @State private var isShowingCamera = false

    var body: some View {
        VStack {
            if let image = image {
                Image(uiImage: image)
                    .resizable()
                    .aspectRatio(contentMode: .fit)
            }

            Button("Take Live Photo") {
                isShowingCamera = true
            }
        }
        .sheet(isPresented: $isShowingImagePicker, onDismiss: loadImagePicker) {
            ImagePicker(sourceType: .photoLibrary) { selectedImage in
                image = selectedImage
            }
        }
        .sheet(isPresented: $isShowingCamera, onDismiss: loadImagePicker) {
            ImagePicker(sourceType: .camera) { capturedImage in
                image = capturedImage
            }
        }
    }

    func loadImagePicker() {
        isShowingImagePicker = false
        isShowingCamera = false
    }
}

struct ImagePicker: UIViewControllerRepresentable {
    typealias UIViewControllerType = UIImagePickerController
    typealias Coordinator = ImagePickerCoordinator

    let sourceType: UIImagePickerController.SourceType
    let completionHandler: (UIImage?) -> Void

    func makeCoordinator() -> Coordinator {
        Coordinator(completionHandler: completionHandler)
    }

    func makeUIViewController(context: Context) -> UIImagePickerController {
        let imagePicker = UIImagePickerController()
        imagePicker.sourceType = sourceType
        imagePicker.delegate = context.coordinator
        return imagePicker
    }

    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {
        // Empty implementation
    }
}

class ImagePickerCoordinator: NSObject, UINavigationControllerDelegate, UIImagePickerControllerDelegate {
    let completionHandler: (UIImage?) -> Void

    init(completionHandler: @escaping (UIImage?) -> Void) {
        self.completionHandler = completionHandler
    }

    func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]) {
        if let selectedImage = info[.originalImage] as? UIImage {
            completionHandler(selectedImage)
        } else {
            completionHandler(nil)
        }
        picker.dismiss(animated: true)
    }

    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        completionHandler(nil)
        picker.dismiss(animated: true)
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
