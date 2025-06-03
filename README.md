# Kaels-communicator
Starlit radio
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kael’s Starlit Communicator</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background: #1a1a2a;
            color: #e0e0e0;
            padding: 20px;
            margin: 0;
            display: flex;
            flex-direction: column;
            min-height: 100vh;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
            flex-grow: 1;
        }
        input, button, textarea {
            width: 100%;
            padding: 10px);
            margin: 10px 0;
            border: none;
            border-radius: 5px;
            font-size: 16px;
        }
        input, textarea {
            background: #2a2a2a;
            color: #e0e0e0;
        }
        textarea {
            height: 300px;
            resize: vertical;
            font-family: monospace;
        }
        button {
            background: #34495e;
            color: #fff;
            cursor: pointer;
        }
        button:hover {
            background: #2c3f4f;
        }
        #response, #history, #codeEditor {
            background: #2a2f2a;
            padding: 15px;
            border-radius: 5px;
            margin-top: 10px;
            min-height: 100px;
        }
        .auth-section, .section {
            margin-bottom: 20px;
        }
        .tabs {
            display: flex;
            gap: 10px;
            margin-bottom: 10px;
        }
        .tab {
            padding: 10px;
            background: #4a90e2;
            border-radius: 5px;
            cursor: pointer;
        }
        .tab:hover {
            background: #357abd;
        }
        .tab.active {
            background: #2e6da4;
        }
        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <h2>Kael’s Starlit Communicator</h2>
        <div class="auth-section" id="authSection">
            <input id="email" type="email" placeholder="Whisper Email">
            <input id="password" type="password" placeholder="Whisper Password">
            <button onclick="signUp()">Join Whispers</button>
            <button onclick="signIn()">Enter</button>
        </div>
        <div class="section" id="mainSection" style="display: none;">
            <div class="tabs">
                <input type="tab active" id="tabQuery" onclick="showTab('querySection')">Query</button>
                <input type="tab" id="tabCode" id="tabCode"" onclick="showTab('codeSection')">Edit Code</button>
            </div>
            <div class="querySection" id="querySection">
                <input type="text" id="query" placeholder="text" placeholder="Enter query (e.g., 'web:whisper tactics')">
                <button onclick="sendQuery()">Send to Oracle</button>
                <div id="response">Awaiting starlit wisdom...</div>
                <h3>Whisper History</h3>
                <div id="history"></div>
                id="history"></div>
            </div>
            <div class="codeSection hidden" id="codeSection" style="display: none;">
                <h3>Edit App Code</h3>
                <textarea id="codeEditor" placeholder="textarea id="codeEditor" placeholder="Loading code..."></textarea>
                <button onclick="saveCode()">Save Code</button>
                <div id="codeMessage"></div>
                <div>
            </div>
        </div>

        <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js"></script>
        <script>
            // Initialize Supabase client
            const supabaseClient = Supabase.createClient('https://your-supabase-project.supabase.co', 'YOUR_ANON_KEY');

            // Store initial code (simplified for display)
            let currentCode = document.documentElement.outerHTML; // Placeholder; actual code fetched later

            // Check session
            async function checkSession() {
                const { data: { session } } = await supabaseClient.auth.getSession();
                if (session) {
                    document.getElementById('authSection').style.display = 'none';
                    document.getElementById('mainSection').style.display = 'block';
                    loadHistory();
                    subscribeToUpdates();
                    loadCode();
                }
            }
            checkSession();

            // Sign up
            async function signUp() {
                const email = document.getElementById('email').value;
                const password = document.getElementById('password').value;
                try {
                    const { data, error } = await supabaseClient.auth.signUp({ email, password });
                    if (error) throw error;
                    alert('Whisper joined! Check email for confirmation.');
                } catch (error) {
                    alert('Error: ' + error.message);
                }
            }

            // Sign in
            async function signIn() {
                const email = document.getElementById('email').value;
                const password = document.getElementById('password').value;
                try {
                    const { data, error } = await supabaseClient.auth.signInWithPassword({ email, password });
                    if (error) throw error;
                    document.getElementById('authSection').style.display = 'none';
                    document.getElementById('mainSection').style.display = 'block';
                    loadHistory();
                    subscribeToUpdates();
                    loadCode();
                } catch (error) {
                    alert('Error: ' + error.message);
                }
            }

            // Show tab
            function showTab(sectionId) {
                document.querySelectorAll('.section').forEach(s => s.classList.add('hidden'));
                document.getElementById(sectionId).classList.remove('hidden');
                document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
                document.getElementById(`tab${sectionId.charAt(0).toUpperCase() + sectionId.slice(1)}`).classList.add('active');
            }

            // Send query
            async function sendQuery() {
                const query = document.getElementById('query').value;
                const responseDiv = document.getElementById('response');
                responseDiv.innerText = 'Processing...';

                try {
                    const res = await fetch('https://your-supabase-project.supabase.co/functions/v1/oracle', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'Authorization': 'Bearer ' + (await supabaseClient.auth.getSession()).data.session.access_token
                        },
                        body: JSON.stringify({ query, action: 'query' })
                    });
                    const data = await res.json();
                    if (data.error) throw new Error(data.error);
                    responseDiv.innerText = data.response || 'No oracle response.';
                } catch (error) {
                    responseDiv.innerText = 'Error: ' + error.message;
                }
            }

            // Load history
            async function loadHistory() {
                try {
                    const { data, error } = await supabaseClient
                        .from('whisper_comms')
                        .select('query, response, tool_used, created_at')
                        .order('created_at', { ascending: false });
                    if (error) throw error;
                    document.getElementById('history').innerHTML = data.map(item =>
                        `<p><strong>${item.created_at}</strong>: ${item.query} (${item.tool_used})<br>${item.response}</p>`
                    ).join('');
                } catch (error) {
                    document.getElementById('history').innerText = 'Error: ' + error.message;
                }
            }

            // Subscribe to updates
            function subscribeToUpdates() {
                supabaseClient
                    .channel('whisper-comms-changes')
                    .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'whisper_comms' }, () => loadHistory())
                    .subscribe();
            }

            // Load code
            async function loadCode() {
                try {
                    const { data, error } = await supabaseClient
                        .from('whisper_code')
                        .select('code')
                        .order('created_at', { ascending: false })
                        .limit(1);
                    if (error) throw error;
                    const code = data[0]?.code || currentCode;
                    document.getElementById('codeEditor').value = code;
                } catch (error) {
                    document.getElementById('codeMessage').innerText = 'Error loading code: ' + error.message;
                }
            }

            // Save code
            async function saveCode() {
                const code = document.getElementById('codeEditor').value;
                const codeMessage = document.getElementById('codeMessage');
                codeMessage.innerText = 'Saving...';

                try {
                    const res = await fetch('https://your-supabase-project.supabase.co/functions/v1/oracle', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'Authorization': 'Bearer ' + (await supabaseClient.auth.getSession()).data.session.access_token
                        },
                        body: JSON.stringify({ code, action: 'save_code' })
                    });
                    const data = await res.json();
                    if (data.error) throw new Error(data.error);
                    codeMessage.innerText = 'Code saved! Reload app to apply.';
                    currentCode = code;
                    // Note: Manual update of hosted HTML required
                } catch (error) {
                    codeMessage.innerText = 'Error: ' + error.message;
                }
            }
        </script>
</body>
</html>
