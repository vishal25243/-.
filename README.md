 
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PeerPay - P2P Payment App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Add the QR code scanning library -->
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
    <style>
        .device-screen {
            border-radius: 24px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
            overflow: hidden;
            transition: all 0.3s ease;
        }
        
        .payment-success {
            animation: successPulse 1.5s ease-in-out;
        }
        
        @keyframes successPulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }
        
        .qr-code {
            background-color: white;
            padding: 16px;
            border-radius: 12px;
            display: inline-block;
        }
        
        .connection-status {
            display: inline-flex;
            align-items: center;
            padding: 6px 12px;
            border-radius: 20px;
            font-size: 14px;
            font-weight: 500;
        }
        
        .status-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            margin-right: 6px;
        }
        
        .transfer-animation {
            position: absolute;
            width: 60px;
            height: 60px;
            background: linear-gradient(135deg, #10b981, #34d399);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            animation: transferMove 1.5s ease-in-out;
            z-index: 10;
        }
        
        @keyframes transferMove {
            0% { 
                transform: translateX(0) scale(1); 
                opacity: 1; 
            }
            100% { 
                transform: translateX(300px) scale(0.5); 
                opacity: 0; 
            }
        }
        
        .tab-button {
            padding: 12px 0;
            width: 50%;
            text-align: center;
            border-bottom: 3px solid transparent;
            transition: all 0.2s;
        }
        
        .tab-button.active {
            border-bottom-color: #4f46e5;
            color: #4f46e5;
            font-weight: 600;
        }
        
        /* QR Scanner Styles */
        #qr-scanner-container {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.9);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 1000;
            display: none;
        }
        
        #qr-scanner {
            width: 80%;
            max-width: 400px;
            position: relative;
        }
        
        #qr-scanner-close {
            position: absolute;
            top: 20px;
            right: 20px;
            color: white;
            background: rgba(255, 255, 255, 0.2);
            border-radius: 50%;
            width: 40px;
            height: 40px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            z-index: 1001;
        }
        
        .scan-area {
            border: 2px solid #4f46e5;
            border-radius: 12px;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 200px;
            height: 200px;
            z-index: 10;
        }
        
        .scan-line {
            height: 2px;
            background: #4f46e5;
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            animation: scanLine 2s linear infinite;
        }
        
        @keyframes scanLine {
            0% { top: 0; }
            100% { top: 100%; }
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen">
    <!-- QR Code Scanner Container -->
    <div id="qr-scanner-container">
        <div id="qr-scanner-close">
            <i class="fas fa-times text-xl"></i>
        </div>
        <div id="qr-scanner"></div>
        <div class="scan-area">
            <div class="scan-line"></div>
        </div>
        <p class="text-white mt-6 text-center">Point your camera at the QR code to connect devices</p>
    </div>
    
    <div class="container mx-auto px-4 py-8">
        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-bold text-gray-800 mb-2">PeerPay</h1>
            <p class="text-gray-600">Send and receive payments directly between devices</p>
        </header>
        
        <!-- Device Selection Tabs -->
        <div class="flex border-b border-gray-200 mb-8">
            <div class="tab-button active" id="device1-tab">
                <i class="fas fa-mobile-alt mr-2"></i>Device 1 (Sender)
            </div>
            <div class="tab-button" id="device2-tab">
                <i class="fas fa-mobile-alt mr-2"></i>Device 2 (Receiver)
            </div>
        </div>
        
        <div class="flex flex-col lg:flex-row gap-8 items-start justify-center">
            <!-- Device 1 (Sender) -->
            <div class="device-screen bg-white p-6 w-full lg:w-96" id="device1">
                <div class="flex items-center justify-between mb-6">
                    <h2 class="text-xl font-semibold text-gray-800">Sender Device</h2>
                    <div class="connection-status bg-red-100 text-red-800">
                        <div class="status-dot bg-red-500"></div>
                        Disconnected
                    </div>
                </div>
                
                <div class="bg-blue-50 p-4 rounded-lg mb-6">
                    <div class="flex justify-between items-center mb-2">
                        <span class="text-gray-600">Balance</span>
                        <span class="text-blue-600 font-medium">$1,250.00</span>
                    </div>
                    <div class="flex justify-between items-center">
                        <span class="text-gray-600">Account</span>
                        <span class="text-gray-800 font-medium">•••• 4582</span>
                    </div>
                </div>
                
                <div id="device1-connect" class="mb-6 text-center">
                    <p class="text-gray-600 mb-4">Connect to receiver device to send payment</p>
                    <div class="qr-code mb-4">
                        <img src="https://storage.googleapis.com/workspace-0f70711f-8b4e-4d94-86f1-2a93ccde5887/image/58b3f916-774d-40ee-a3fb-0520888ac608.png" alt="QR code for Device 1 connection with blue background and white text" class="mx-auto">
                    </div>
                    <p class="text-sm text-gray-500 mb-4">Scan this QR code with the receiver device</p>
                    <button class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 rounded-lg font-medium transition-colors" id="connect-device1">
                        <i class="fas fa-link mr-2"></i> Connect Manually
                    </button>
                </div>
                
                <div id="device1-send" class="hidden">
                    <div class="mb-6">
                        <label class="block text-gray-700 mb-2">Send to</label>
                        <div class="bg-gray-100 p-3 rounded-lg">
                            <div class="flex items-center">
                                <div class="w-10 h-10 bg-green-100 rounded-full flex items-center justify-center mr-3">
                                    <i class="fas fa-user text-green-600"></i>
                                </div>
                                <div>
                                    <div class="font-medium">Device 2 (Receiver)</div>
                                    <div class="text-sm text-gray-500">Connected</div>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <div class="mb-6">
                        <label class="block text-gray-700 mb-2">Amount</label>
                        <div class="relative">
                            <span class="absolute left-3 top-3 text-gray-500">$</span>
                            <input type="number" class="w-full pl-8 p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500" placeholder="0.00" min="0" step="0.01" id="send-amount">
                        </div>
                    </div>
                    
                    <div class="mb-6">
                        <label class="block text-gray-700 mb-2">Note (optional)</label>
                        <input type="text" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500" placeholder="Dinner payment" id="send-note">
                    </div>
                    
                    <button id="sendButton" class="w-full bg-blue-600 hover:bg-blue-700 text-white py-3 rounded-lg font-medium transition-colors flex items-center justify-center">
                        <i class="fas fa-paper-plane mr-2"></i> Send Payment
                    </button>
                </div>
            </div>
            
            <!-- Connection Visualization -->
            <div class="hidden lg:flex flex-col items-center justify-center w-32 my-8 relative">
                <div class="w-1 h-32 bg-blue-200 rounded-full"></div>
                <div class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-white p-2 rounded-full shadow-md">
                    <i class="fas fa-wifi text-blue-500 text-xl"></i>
                </div>
                <div id="transferAnimation" class="transfer-animation opacity-0">
                    <i class="fas fa-dollar-sign text-xl"></i>
                </div>
            </div>
            
            <!-- Device 2 (Receiver) -->
            <div class="device-screen bg-white p-6 w-full lg:w-96 hidden" id="device2">
                <div class="flex items-center justify-between mb-6">
                    <h2 class="text-xl font-semibold text-gray-800">Receiver Device</h2>
                    <div class="connection-status bg-red-100 text-red-800">
                        <div class="status-dot bg-red-500"></div>
                        Disconnected
                    </div>
                </div>
                
                <div class="bg-gray-50 p-4 rounded-lg mb-6">
                    <div class="flex justify-between items-center">
                        <span class="text-gray-600">Balance</span>
                        <span class="text-gray-800 font-medium">$850.00</span>
                    </div>
                </div>
                
                <div id="device2-connect" class="text-center">
                    <p class="text-gray-600 mb-4">Connect to sender device to receive payments</p>
                    <div class="qr-code mb-4">
                        <img src="https://storage.googleapis.com/workspace-0f70711f-8b4e-4d94-86f1-2a93ccde5887/image/de372a34-79a7-4977-8013-3fb24968a9b0.png" alt="QR code for Device 2 connection with green background and white text" class="mx-auto">
                    </div>
                    <div class="flex gap-3">
                        <button class="flex-1 bg-gray-200 hover:bg-gray-300 text-gray-800 py-2 rounded-lg font-medium transition-colors" id="scan-qr-btn">
                            <i class="fas fa-camera mr-2"></i> Scan QR
                        </button>
                        <button class="flex-1 bg-green-600 hover:bg-green-700 text-white py-2 rounded-lg font-medium transition-colors" id="connect-device2">
                            <i class="fas fa-link mr-2"></i> Connect
                        </button>
                    </div>
                </div>
                
                <div id="device2-receive" class="hidden">
                    <div class="text-center py-4">
                        <div class="text-gray-400 mb-4">
                            <i class="fas fa-inbox text-4xl"></i>
                        </div>
                        <p class="text-gray-500">Waiting for incoming payment...</p>
                    </div>
                </div>
                
                <div id="receivedPayment" class="hidden">
                    <div class="bg-green-50 border border-green-200 rounded-lg p-4 mb-6">
                        <div class="flex justify-between items-center mb-2">
                            <span class="text-green-800 font-medium">Received</span>
                            <span class="text-green-800 font-medium">+$<span id="receivedAmount">0.00</span></span>
                        </div>
                        <div class="text-sm text-green-700">
                            From: <span id="senderInfo">Sender Device</span>
                        </div>
                        <div class="text-sm text-green-700 mt-1">
                            Note: <span id="paymentNote">Payment note</span>
                        </div>
                    </div>
                    
                    <div class="bg-gray-50 p-4 rounded-lg mb-6">
                        <div class="flex justify-between items-center">
                            <span class="text-gray-600">New Balance</span>
                            <span class="text-gray-800 font-medium">$<span id="newBalance">850.00</span></span>
                        </div>
                    </div>
                    
                    <button class="w-full bg-green-600 hover:bg-green-700 text-white py-3 rounded-lg font-medium transition-colors">
                        <i class="fas fa-check-circle mr-2"></i> Done
                    </button>
                </div>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // UI Elements
            const device1Tab = document.getElementById('device1-tab');
            const device2Tab = document.getElementById('device2-tab');
            const device1 = document.getElementById('device1');
            const device2 = document.getElementById('device2');
            const device1Connect = document.getElementById('device1-connect');
            const device1Send = document.getElementById('device1-send');
            const device2Connect = document.getElementById('device2-connect');
            const device2Receive = document.getElementById('device2-receive');
            const receivedPayment = document.getElementById('receivedPayment');
            const sendButton = document.getElementById('sendButton');
            const connectDevice1 = document.getElementById('connect-device1');
            const connectDevice2 = document.getElementById('connect-device2');
            const scanQrBtn = document.getElementById('scan-qr-btn');
            const qrScannerContainer = document.getElementById('qr-scanner-container');
            const qrScannerClose = document.getElementById('qr-scanner-close');
            const transferAnimation = document.getElementById('transferAnimation');
            
            // QR Code Scanner
            let html5QrCode = null;
            let isScanning = false;
            
            // State
            let devicesConnected = false;
            
            // Tab Switching
            device1Tab.addEventListener('click', () => {
                device1Tab.classList.add('active');
                device2Tab.classList.remove('active');
                device1.classList.remove('hidden');
                device2.classList.add('hidden');
            });
            
            device2Tab.addEventListener('click', () => {
                device2Tab.classList.add('active');
                device1Tab.classList.remove('active');
                device2.classList.remove('hidden');
                device1.classList.add('hidden');
            });
            
            // Connection Simulation
            connectDevice1.addEventListener('click', connectDevices);
            connectDevice2.addEventListener('click', connectDevices);
            
            // QR Code Scanner
            scanQrBtn.addEventListener('click', startQrCodeScan);
            qrScannerClose.addEventListener('click', stopQrCodeScan);
            
            function connectDevices() {
                if (devicesConnected) return;
                
                devicesConnected = true;
                
                // Update connection status on both devices
                const statusIndicators = document.querySelectorAll('.connection-status');
                statusIndicators.forEach(el => {
                    el.className = 'connection-status bg-green-100 text-green-800';
                    el.innerHTML = '<div class="status-dot bg-green-500"></div> Connected';
                });
                
                // Show send interface on device 1
                device1Connect.classList.add('hidden');
                device1Send.classList.remove('hidden');
                
                // Show receive interface on device 2
                device2Connect.classList.add('hidden');
                device2Receive.classList.remove('hidden');
            }
            
            function startQrCodeScan() {
                // Show the scanner
                qrScannerContainer.style.display = 'flex';
                
                // Initialize the scanner if needed
                if (!html5QrCode) {
                    html5QrCode = new Html5Qrcode("qr-scanner");
                }
                
                // Start scanning
                Html5Qrcode.getCameras().then(cameras => {
                    if (cameras && cameras.length) {
                        // Use the first camera (usually the back camera)
                        const cameraId = cameras[0].id;
                        
                        html5QrCode.start(
                            cameraId,
                            {
                                fps: 10,
                                qrbox: { width: 200, height: 200 }
                            },
                            qrCodeMessage => {
                                // QR code detected
                                console.log("QR Code detected:", qrCodeMessage);
                                stopQrCodeScan();
                                
                                // Check if the QR code is valid (in a real app you'd verify this)
                                if (qrCodeMessage.includes("Device 1")) {
                                    connectDevices();
                                } else {
                                    alert("Invalid QR code. Please scan the QR code from Device 1.");
                                }
                            },
                            errorMessage => {
                                // Ignore scanning errors (they happen often while scanning)
                            }
                        ).catch(err => {
                            console.error("Unable to start scanning", err);
                            alert("Unable to start camera: " + err);
                            stopQrCodeScan();
                        });
                        
                        isScanning = true;
                    } else {
                        alert("No camera found on this device.");
                    }
                }).catch(err => {
                    console.error("Error getting cameras:", err);
                    alert("Error accessing camera: " + err);
                });
            }
            
            function stopQrCodeScan() {
                if (html5QrCode && isScanning) {
                    html5QrCode.stop().then(() => {
                        isScanning = false;
                        qrScannerContainer.style.display = 'none';
                    }).catch(err => {
                        console.error("Failed to stop scanning", err);
                    });
                } else {
                    qrScannerContainer.style.display = 'none';
                }
            }
            
            // Payment sending
            sendButton.addEventListener('click', function() {
                if (!devicesConnected) {
                    alert('Please connect to a device first');
                    return;
                }
                
                const amountInput = document.getElementById('send-amount');
                const noteInput = document.getElementById('send-note');
                
                const amount = amountInput.value;
                const note = noteInput.value;
                
                if (!amount || amount <= 0) {
                    alert('Please enter a valid amount');
                    return;
                }
                
                // Disable button during transfer
                sendButton.disabled = true;
                sendButton.textContent = 'Sending...';
                
                // Show transfer animation
                transferAnimation.classList.remove('opacity-0');
                transferAnimation.style.animation = 'none';
                setTimeout(() => {
                    transferAnimation.style.animation = 'transferMove 1.5s ease-in-out';
                }, 10);
                
                // Simulate transfer delay
                setTimeout(() => {
                    // Reset animation
                    transferAnimation.classList.add('opacity-0');
                    
                    // Update receiver UI
                    document.getElementById('receivedAmount').textContent = parseFloat(amount).toFixed(2);
                    document.getElementById('paymentNote').textContent = note || 'No note provided';
                    document.getElementById('newBalance').textContent = (850 + parseFloat(amount)).toFixed(2);
                    
                    // Show received payment on device 2
                    device2Receive.classList.add('hidden');
                    receivedPayment.classList.remove('hidden');
                    
                    // Reset sender form
                    amountInput.value = '';
                    noteInput.value = '';
                    
                    // Re-enable button
                    sendButton.disabled = false;
                    sendButton.innerHTML = '<i class="fas fa-paper-plane mr-2"></i> Send Payment';
                    
                    // Add success animation to receiver device
                    device2.classList.add('payment-success');
                    setTimeout(() => {
                        device2.classList.remove('payment-success');
                    }, 1500);
                    
                    // Switch to device 2 view
                    device2Tab.click();
                }, 1600);
            });
        });
    </script>
</body>
</html>
