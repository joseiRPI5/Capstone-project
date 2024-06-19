
<!DOCTYPE html>
<html lang="en">
<head>

     <meta charset="UTF-8">
     <title>webRTC Sender</title>
     <style>
        body {
             font-family: Arial, sans-serif;
             text-align: center;
             margin: 0;
             paddings: 0;
            }

          .video-container {
               display: flex;
               justify content: center;
               margin-top: 20px;
              }

            video {
               width: 45%;
               margin: 10px;
               border: 1px solid black;
               border-radius: 5px;
             }

             button:hover {
                 background-color: #0056b3;
              }

             button {
                margin-top: 20px;
                padding: 10px 20px;
                font-size: 16px;
                cursor: pointer;
                background-color: #007bff;
                color: white;
                border: none;
                border-radius: 5px;
           }
        </style>
        <!-- Ensure Strophe.js is included correctly -->
        <script src="https://cdnjs.cloudflare.com/ajax/libs/strophe.js/3.0.0/strophe.min.js"></script>
        <script>

              document.addEventListener("DOMContentLoaded", () => {
                   let localStream;
                   let peerConnection;
                   const servers = { iceServers: [{ urls: "stun:stun.l.google.com:19302" }] };
                   const xmppServerUrl = "wss://yourdomain.com/xmpp-websocket";
                   const xmppUser = "user@yourdomain.com";
                   const xmppPassword = "badcompany";
                   const xmppDomain = "yourdomain.com";



                   const connection = new Strophe.Connection(xmppServerUrl);
                   connection.connect(xmppUser, xmppPassword, onConnect);

                   function onConnect(status) {
                       if (status === Strophe.Status.CONNECTED) {
                          console.log("Connected to XMPP server");
                          document.getElementById("startButton").disabled = false;
                        }
                     }

                 async function startCall() {
                     try {
                         localStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
                         document.getElementById("localVideo").srcObject = localStream;


                         peerConnection = new RTCPeerConnection(servers);
                         peerConnection.onicecandidate = handleICECandidateEvent;
                         peerConnection.onnegotiationneeded = handleNegotiationNeededEvent;

                         localStream.getTracks().forEach(track => peerConnection.addTrack(track , localStream));

                         document.getElementById("startButton").disabled = true;
                         document.getElementById("endButton").disabled = false;
                      } catch (error) {
                          console.error("Error accessing media devices:", error);
                      }
                    }

                    function endCall() {
                      if (peerConnection) {
                          peerConnection.close();
                          connection.disconnect();
                          localStream.getTracks().forEach(track => track.stop());
                          document.getElementById("localVideo").srcObject = null;
                          peerConnection = null;
                          document.getElementById("startButton").disabled = false;
                          document.getElementById("endButton").disabled = true;
                        }
                      }

                      function handleICECandidateEvent(event) {
                        if (event.candidate) {
                            sendMessage({ type: "candidate", candidate: event.candidate });
                         }
                      }

                       async function handleNegotiationNeededEvent() {
                            try {
                                const offer = await peerConnection.createOffer();
                                await peerConnection.setLocalDescription(offer);
                                sendMessage({ type: "offer", sdp: peerConnection.localDescription });

                          } catch (error) {
                              console.error("Error creating offer:", error);
                           }
                        }

                       function sendMessage(message) {

     const msg = $msg({ to: "peer@yourdomain.com", type: "chat" }).c("body").t(JSON.stringify(message));
                             connection.send(msg);
                        }

                        // Attach event listeners to the buttons
                        document.getElementById("startButton").onclick = startCall;
                        document.getElementById("endButton").onclick = endCall;
                     });

                  </script>
              </head>
              <body>

                  <h1>webRTC Sender</h1>
                  <div id="controls">
                         <button id="startButton" disabled>Start Call</button>
                         <button id="endButton" disabled>End Call</button>
                  </div>
                  <video id="localVideo" autoplay muted></video>


               </body>
               </html>

