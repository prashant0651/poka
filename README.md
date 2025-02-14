<?php
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;
require 'vendor/autoload.php';
require 'fpdf.php';

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $name = $_POST['name'];
    $email = $_POST['email'];
    $service = $_POST['service'];
    $govtService = $_POST['govtService'];
    $referenceID = uniqid("NB-");
    
    // Handle file upload
    $uploadDir = 'uploads/';
    $uploadedFile = '';
    if (!empty($_FILES['documentUpload']['name'])) {
        $fileName = basename($_FILES['documentUpload']['name']);
        $targetFilePath = $uploadDir . $fileName;
        if (move_uploaded_file($_FILES['documentUpload']['tmp_name'], $targetFilePath)) {
            $uploadedFile = $targetFilePath;
        }
    }
    
    // Generate PDF Receipt
    class PDF extends FPDF {
        function Header() {
            $this->SetFont('Arial','B',12);
            $this->Cell(0,10,'Neonbyte Web Hosting Company Ltd.',0,1,'C');
            $this->Ln(10);
        }
    }
    
    $pdf = new PDF();
    $pdf->AddPage();
    $pdf->SetFont('Arial','B',16);
    $pdf->Cell(40,10,'Receipt');
    $pdf->Ln(10);
    $pdf->SetFont('Arial','',12);
    $pdf->Cell(40,10,'Reference ID: ' . $referenceID);
    $pdf->Ln(10);
    $pdf->Cell(40,10,'Name: ' . $name);
    $pdf->Ln(10);
    $pdf->Cell(40,10,'Email: ' . $email);
    $pdf->Ln(10);
    $pdf->Cell(40,10,'Service: ' . $service);
    $pdf->Ln(10);
    $pdf->Cell(40,10,'Government Service: ' . $govtService);
    $pdf->Ln(10);
    
    // QR Code Generation (Static URL for demo, modify as needed)
    $qrData = "Reference ID: $referenceID\nName: $name\nEmail: $email\nService: $service\nGovernment Service: $govtService";
    $qrCodePath = "qrcodes/" . $referenceID . ".png";
    QRcode::png($qrData, $qrCodePath, QR_ECLEVEL_L, 4);
    $pdf->Image($qrCodePath, 10, 100, 50, 50);
    
    // Save PDF
    $pdfPath = 'receipts/' . $referenceID . '.pdf';
    $pdf->Output($pdfPath, 'F');
    
    // Send Email with Attachment
    $mail = new PHPMailer(true);
    try {
        $mail->setFrom('prashantkumarsingh065@gmail.com', 'Neonbyte Web Hosting');
        $mail->addAddress('prashantkumarsingh065@gmail.com');
        $mail->addAddress($email);
        $mail->Subject = "Order Confirmation - $referenceID";
        $mail->Body = "Thank you for your order! Your Reference ID is $referenceID.";
        $mail->addAttachment($pdfPath);
        if ($uploadedFile) {
            $mail->addAttachment($uploadedFile);
        }
        $mail->send();
        echo "Order successfully placed! Check your email for the receipt.";
    } catch (Exception $e) {
        echo "Error: " . $mail->ErrorInfo;
    }
}
?>
