import cv2
import webbrowser
import sys

APP_NAME = "ScanGuard"
app_running = True


def open_camera():
    print("=" * 50)
    print("SEARCHING FOR CAMERA...")
    print("=" * 50)

    cap = cv2.VideoCapture(0)
    if not cap.isOpened():
        print("❌ Camera not found")
        sys.exit(1)

    print("✓ Camera opened successfully")
    return cap


def main():
    global app_running

    print("\nSTARTING SCANGUARD")
    print("Press 'Q' to quit\n")

    cap = open_camera()
    qr_detector = cv2.QRCodeDetector()

    while app_running:
        ret, frame = cap.read()
        if not ret:
            continue

        data, bbox, _ = qr_detector.detectAndDecode(frame)

        if data:
            print("\nQR CODE DETECTED")
            print("Data:", data)

            # Draw bounding box
            if bbox is not None:
                bbox = bbox.astype(int)
                for i in range(len(bbox[0])):
                    pt1 = tuple(bbox[0][i])
                    pt2 = tuple(bbox[0][(i + 1) % len(bbox[0])])
                    cv2.line(frame, pt1, pt2, (0, 255, 0), 2)

            # Ask user before opening
            choice = input("Do you want to open this link? (y/n): ").lower()

            if choice == "y" and data.startswith(("http://", "https://")):
                print("Opening link...")
                webbrowser.open(data)
            else:
                print("Link NOT opened")

        cv2.imshow(APP_NAME, frame)

        if cv2.waitKey(1) & 0xFF == ord("q"):
            break

    cap.release()
    cv2.destroyAllWindows()
    print("Goodbye!")


if __name__ == "__main__":
    main()
