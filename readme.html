<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Project Overview: Cross-Platform Clipboard Sync</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc;
        }
        .container {
            max-width: 900px;
            margin: auto;
            padding: 2rem;
        }
        .card {
            background-color: white;
            border-radius: 0.75rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
            margin-bottom: 2rem;
            overflow: hidden;
        }
        .card-header {
            background-color: #f1f5f9;
            padding: 1rem 1.5rem;
            border-bottom: 1px solid #e2e8f0;
            font-weight: 600;
            font-size: 1.25rem;
            display: flex;
            align-items: center;
        }
        .card-header svg {
            margin-right: 0.75rem;
            width: 24px;
            height: 24px;
        }
        .card-body {
            padding: 1.5rem;
        }
        h3 {
            font-size: 1.1rem;
            font-weight: 600;
            margin-top: 1.5rem;
            margin-bottom: 0.5rem;
            color: #1e293b;
            border-bottom: 1px solid #e2e8f0;
            padding-bottom: 0.25rem;
        }
        ul {
            list-style-type: none;
            padding-left: 0;
        }
        li {
            position: relative;
            padding-left: 1.75rem;
            margin-bottom: 0.75rem;
            color: #475569;
        }
        li::before {
            content: '✓';
            position: absolute;
            left: 0;
            color: #22c55e;
            font-weight: bold;
        }
        li.detail::before {
            content: '💡';
            color: #f59e0b;
        }
        li.bug::before {
            content: '🐞';
            color: #ef4444;
        }
        code {
            background-color: #e2e8f0;
            color: #334155;
            padding: 0.2rem 0.4rem;
            border-radius: 0.25rem;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <div class="container">
        <header class="text-center mb-12">
            <h1 class="text-4xl font-bold text-slate-800">Project: Seamless Clipboard Sync</h1>
            <p class="text-slate-500 mt-2">A high-level overview of the architecture, tasks, and potential challenges for a secure, cross-platform clipboard utility.</p>
        </header>

        <!-- Core Architecture -->
        <div class="card">
            <div class="card-header">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M10.5 6a7.5 7.5 0 1 0 7.5 7.5h-7.5V6Z" /><path stroke-linecap="round" stroke-linejoin="round" d="M13.5 10.5H21A7.5 7.5 0 0 0 13.5 3v7.5Z" /></svg>
                Core Architecture
            </div>
            <div class="card-body">
                <p class="text-slate-600">The system will use a client-server model. A central server is required to reliably route messages between devices, which will be behind different firewalls and NATs. Peer-to-peer communication over the internet is not practical for this use case.</p>
                <h3>Key Principles</h3>
                <ul>
                    <li><strong>Central WebSocket Server:</strong> Acts as a real-time message broker. Clients (Mac/Android apps) maintain a persistent connection to the server, allowing for instant data push.</li>
                    <li><strong>Stateless Pairing:</strong> No traditional user accounts. Devices are paired using a temporary, unique, human-readable code. The server's only job is to map device IDs together.</li>
                    <li><strong>End-to-End Encryption (E2EE):</strong> The most critical component. Clipboard content is encrypted on the source device and can *only* be decrypted by a paired device. The server will never see the unencrypted data.</li>
                </ul>
            </div>
        </div>

        <!-- Backend Server -->
        <div class="card">
            <div class="card-header">
                 <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M21.75 17.25v-.228a4.5 4.5 0 0 0-.12-1.03l-2.268-9.64a3.375 3.375 0 0 0-3.285-2.65H8.25a3.375 3.375 0 0 0-3.285 2.65l-2.268 9.64a4.5 4.5 0 0 0-.12 1.03v.228m15.45-1.5H6.25m15.45 0v7.5a2.25 2.25 0 0 1-2.25 2.25H8.25a2.25 2.25 0 0 1-2.25-2.25v-7.5m15.45 0c0-1.657-1.343-3-3-3H8.25c-1.657 0-3 1.343-3 3m15 0c0 .414-.336.75-.75.75H5.25c-.414 0-.75-.336-.75-.75m15.5 0v-1.372c0-.828-.672-1.5-1.5-1.5H6.75c-.828 0-1.5.672-1.5 1.5v1.372m14.5 0A3 3 0 0 1 18 19.25v1.5a2.25 2.25 0 0 1-2.25 2.25H8.25a2.25 2.25 0 0 1-2.25-2.25v-1.5a3 3 0 0 1-1.5-2.625v-1.372c0-.828.672-1.5 1.5-1.5h10.5c.828 0 1.5.672 1.5 1.5v1.372Z" /></svg>
                Backend Server (Node.js + WebSockets)
            </div>
            <div class="card-body">
                <h3>Major Tasks</h3>
                <ul>
                    <li>Setup a Node.js project with a WebSocket library (e.g., <code>ws</code>).</li>
                    <li>Implement logic to handle new WebSocket connections and assign a unique ID to each.</li>
                    <li>Create an endpoint/message type for device pairing (linking two connection IDs).</li>
                    <li>Develop the core message routing logic: when an encrypted message is received from one client, forward it to all its paired clients.</li>
                    <li>Deploy the server to a hosting service (e.g., Vercel, Heroku, DigitalOcean).</li>
                </ul>
                <h3 class="text-amber-600">Details Often Ignored</h3>
                <ul>
                    <li class="detail"><strong>Connection State Management:</strong> Keep an in-memory map of connections (e.g., <code>{ "connectionId": { socket, paired_ids: [] } }</code>).</li>
                    <li class="detail"><strong>Graceful Disconnects:</strong> Properly handle the <code>close</code> event on a WebSocket to remove the client from the map and notify paired devices if necessary.</li>
                    <li class="detail"><strong>Scalability:</strong> For a personal project, a single server instance is fine. For a larger app, you'd need a message broker like Redis to handle state across multiple server instances.</li>
                </ul>
                <h3 class="text-red-600">Common Bugs to Watch For</h3>
                <ul>
                    <li class="bug"><strong>Memory Leaks:</strong> Not cleaning up disconnected client data from the connection map.</li>
                    <li class="bug"><strong>Zombie Connections:</strong> Connections that appear open but are actually dead. Implement a heartbeat/ping-pong mechanism to detect and close these.</li>
                    <li class="bug"><strong>Broadcast Loops:</strong> Accidentally sending a message back to the original sender, creating an infinite loop.</li>
                </ul>
            </div>
        </div>

        <!-- macOS App -->
        <div class="card">
            <div class="card-header">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M9 17.25v1.007a3 3 0 0 1-.879 2.122L7.5 21h9l-.621-.621A3 3 0 0 1 15 18.257V17.25m6-12V15a2.25 2.25 0 0 1-2.25 2.25H5.25A2.25 2.25 0 0 1 3 15V5.25m18 0A2.25 2.25 0 0 0 18.75 3H5.25A2.25 2.25 0 0 0 3 5.25m18 0V12a2.25 2.25 0 0 1-2.25 2.25H5.25A2.25 2.25 0 0 1 3 12V5.25" /></svg>
                macOS App (SwiftUI)
            </div>
            <div class="card-body">
                <h3>Major Tasks</h3>
                <ul>
                    <li>Create a new SwiftUI project configured as a Menu Bar App (<code>MenuBarExtra</code>).</li>
                    <li>Implement a background timer to periodically check <code>NSPasteboard.general.changeCount</code>.</li>
                    <li>When <code>changeCount</code> changes, read the clipboard content (start with strings).</li>
                    <li>Implement client-side E2EE: encrypt the content.</li>
                    <li>Establish and maintain a WebSocket connection to the server. Send encrypted data on copy.</li>
                    <li>Listen for incoming messages, decrypt them, and write the content to the pasteboard.</li>
                </ul>
                <h3 class="text-amber-600">Details Often Ignored</h3>
                <ul>
                    <li class="detail"><strong>Efficient Polling:</strong> A 1-second timer is a good starting point. Too fast wastes CPU; too slow feels laggy.</li>
                    <li class="detail"><strong>Duplicate Prevention:</strong> Store the <code>changeCount</code> of the last item you sent. Don't re-send it. Also, when you receive an item and write it to the clipboard, update your local <code>changeCount</code> so you don't immediately detect it as a "new" item and send it back.</li>
                    <li class="detail"><strong>Launch on Login:</strong> Implement functionality to make the app a Login Item so it starts automatically.</li>
                    <li class="detail"><strong>Code Signing:</strong> Even for personal use, signing the app with a paid developer account ($99/year) is highly recommended to bypass Gatekeeper warnings and hassles.</li>
                </ul>
                <h3 class="text-red-600">Common Bugs to Watch For</h3>
                <ul>
                    <li class="bug"><strong>High CPU Usage:</strong> Inefficient clipboard polling or a tight loop in the background thread.</li>
                    <li class="bug"><strong>UI Freezing:</strong> Performing networking or encryption on the main thread. All heavy lifting must be done in the background.</li>
                    <li class="bug"><strong>Notch Issues:</strong> On notched MacBooks, having too many menu bar items can hide your app's icon. The UI needs to be simple.</li>
                    <li class="bug"><strong>Permissions:</strong> The app might be blocked by firewalls (like Little Snitch or the built-in one) without user permission.</li>
                </ul>
            </div>
        </div>

        <!-- Android App -->
        <div class="card">
            <div class="card-header">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M10.5 1.5H8.25A2.25 2.25 0 0 0 6 3.75v16.5a2.25 2.25 0 0 0 2.25 2.25h7.5A2.25 2.25 0 0 0 18 20.25V3.75a2.25 2.25 0 0 0-2.25-2.25H13.5m-3 0V3h3V1.5m-3 0h3m-3 18.75h3" /></svg>
                Android App (Kotlin)
            </div>
            <div class="card-body">
                <h3>Major Tasks</h3>
                <ul>
                    <li>Create a new Android project using Kotlin.</li>
                    <li>Implement a **Foreground Service**. This is non-negotiable for background clipboard access on modern Android.</li>
                    <li>The service must display a persistent notification to the user.</li>
                    <li>In the service, register a <code>ClipboardManager.OnPrimaryClipChangedListener</code>.</li>
                    <li>Implement client-side E2EE, WebSocket connection, and message handling, similar to the macOS app.</li>
                </ul>
                <h3 class="text-amber-600">Details Often Ignored</h3>
                <ul>
                    <li class="detail"><strong>The Foreground Service is Key:</strong> This is the biggest hurdle. The app will not work without it. The user must accept seeing a permanent notification while the app is active.</li>
                    <li class="detail"><strong>Battery Optimization:</strong> Android's "Doze" mode will kill your app's background process and WebSocket connection. You must request the user to grant the <code>REQUEST_IGNORE_BATTERY_OPTIMIZATIONS</code> permission.</li>
                    <li class="detail"><strong>Network State Changes:</strong> The app needs to handle switching between Wi-Fi and mobile data gracefully, re-establishing the WebSocket connection automatically.</li>
                    <li class="detail"><strong>Duplicate Prevention:</strong> Similar to the Mac app, you need a mechanism to avoid re-uploading content that the app itself just wrote to the clipboard. Hashing the content is a simple way to check.</li>
                </ul>
                <h3 class="text-red-600">Common Bugs to Watch For</h3>
                <ul>
                    <li class="bug"><strong>Service being killed by OS:</strong> Aggressive power management on some Android brands (e.g., Xiaomi, OnePlus) can kill the service even if it's a "Foreground Service." There's no perfect solution besides guiding the user to app-specific battery settings.</li>
                    <li class="bug"><strong>Clipboard listener not firing:</strong> Forgetting to re-register the listener if the service restarts.</li>
                    <li class="bug"><strong>UI lag:</strong> Performing networking on the UI thread. Use Coroutines or other async methods.</li>
                </ul>
            </div>
        </div>
    </div>
</body>
</html>
