# Lab 4 - HTTP Server Lab
# Using IntelliJ

At the end of this lab, you should have an HTTP server that is able to serve up two different files to a browser. This lab should work across browsers such as FireFox and Chrome. In addition, you need to also complete the quiz on Canvas named Lab4_Quiz.

This lab builds on the networking and HTTP lectures and gives you experience working with a transport layer protocol (TCP) and implementing basic elements of an application layer protocol (HTTP).

In other words, we are writing HTTP requests and HTTP responses over a TCP socket. Recall how this works: A browser sends an HTTP request over a TCP socket. TCP, in turn, sends messages to the IP layer. The IP layer communicates with the underlying network.

This repository's Example-Socket-Code directory has example client and server socket implementations in several languages. You should browse these files to see how socket connections can be made in several languages. We are not using any of the code found in Example-Socket-Code for this lab.

The starter code that we will use is named StartingWebServer.java and is shown next. Please be sure to read over this code and make sense of it before using it.

```
// Lab 4 - HTTP Server Lab
// We will be making the necessary modifications to this code so that it is able
// to respond to two requests from a browser.
// One request will be for http://localhost:7777/index.html and another request
// will be for http://localhost:7777/test.html.
// This server will also work if the user enters a request for a file that is not available.


import java.net.*;
import java.io.*;

// StartingWebServer is a web server that needs to be modified
// for the HTTP Server Lab.

public class StartingWebServer {

    public static void main(String args[]) {
        Socket clientSocket = null;
        try {
            int serverPort = 7777; // the port that this server will listen on

            // Create a new server socket on the port
            ServerSocket listenSocket = new ServerSocket(serverPort);

            // Handle mulitple visits
            while (true) {

                // block and wait for a visit
                clientSocket = listenSocket.accept();

                // display on the server console
                System.out.println("We have a visit");

                // Set up "inFromSocket" to read from the client socket
                BufferedReader inFromSocket = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));

                // Set up "out" to write to the client socket
                PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(clientSocket.getOutputStream())));

                // read a line of HTTP from the TCP socket
                String data = inFromSocket.readLine();

                // Do we have a GET request
                if (data != null && data.startsWith("GET")) {

                    // extract file name from the request
                    String[] tokens = data.split(" ");
                    String requestedFile = tokens[1].substring(1);

                    // Now requestedFile contains the file name
                    System.out.println("The file is " + requestedFile);
                    BufferedReader fileIn = null;
                    try {
                        // open the file
                        File file = new File(requestedFile);
                        fileIn = new BufferedReader(new FileReader(file));

                        System.out.println("We need to send back response headers now.");

                        System.out.println("We need to send back file data now.");

                        // The next 7 lines are just for startup. Remove
                        // these lines 7 when the lab is complete.
                        System.out.println("This program is not yet complete, so send back an HTTP 404 error.");
                        System.out.println("No such file");
                        out.println("HTTP/1.1 404 Not Found");
                        out.println("Content-Type: text/html");
                        out.println("Connection: close");
                        out.println();
                        out.println("<html><body><h1>File Not Found</h1></body></html>");

                    } catch (FileNotFoundException e) {

                        System.out.println("No such file");
                        out.println("HTTP/1.1 404 Not Found");
                        out.println("Content-Type: text/html");
                        out.println("Connection: close");
                        out.println();
                        out.println("<html><body><h1>File Not Found</h1></body></html>");
                    } finally {
                        if (fileIn != null) {
                            fileIn.close();
                        }
                    }
                }
                System.out.println("Closing ");
                out.flush();
                out.close();
                clientSocket.close();
            }
        } catch (IOException e) {
            System.out.println("IO Exception:" + e.getMessage());
        } finally {
            try {
                if (clientSocket != null) {
                    clientSocket.close();
                }
            } catch (IOException e) {
                System.out.println("IOException ");
            }
        }
    }
}

```
StartingWebServer.java makes no use of JEE or Tomcat. It is a stand alone web server. To create a simple, stand alone Java application in IntelliJ Ultimate, open IntelliJ and select Project/New Project/New Project and then provide a name for a Java class. Right click the project name and select new Java class.

Copy and paste (the unmodified) StartingWebServer.java into IntelliJ and run it. Visit it with a browser by visiting http://localhost:7777/index.html.

:checkered_flag: **Exercise 1. Answer Question 1 in Lab4_Quiz.**

For this lab, you will make the necessary modifications to StartingWebServer.java so that it serves up two files: index.html and test.html.

index.html file is shown here:

```
<!DOCTYPE html>
<html>
<head>
    <title>HTML Tutorial</title>
</head>
<body>

<h1>This is a heading</h1>
<p>This is a paragraph.</p>

</body>
</html>
```

The file test.html is shown next:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Color Changer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
        }
    </style>
</head>
<body>
<h1>Welcome to the Color Changer!</h1>
<p>Click the button below to change the background color.</p>
<button onclick="changeColor()">Change Color</button>

<script>
    function changeColor() {
        const randomColor = '#' + Math.floor(Math.random() * 16777215).toString(16);
        document.body.style.backgroundColor = randomColor;
    }
</script>
</body>
</html>
```
To include these two files in your IntelliJ project, right click the project node and choose "new file". Save each file, in this way.

In this lab, the web browser (e.g. Chrome) will be your HTTP client.

Spend some time studying StartingWebServer.java. With this understanding, it will be much easier to build the web server.

This server handles one request at a time. It is not a multi-threaded server that is able to handle simultaneous visits. In this lab, we will be keeping things simple and will not be using threads.

In Chrome, here is an HTTP request generated by a visit to http://localhost:3000/test.html

```
GET /test.html HTTP/1.1
Host: localhost:3000
Connection: keep-alive
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_2)  ...
Accept-Encoding: gzip,deflate,sdch
Accept-Language: en-US,en;q=0.8
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.3
```
As you can see, the browser provides the HTTP request broken into many components. Here, the browser is attempting to fetch a file named test.html.
The other components are important for a sophisticated web server, such as Apache or Tomcat. In this lab, we are only interested in the first line.

HTTP is primarily based upon four verbs: GET, PUT, POST and DELETE. In this lab, we are only handling GET request. Note how this is detected in StartingWebServer.java.

:checkered_flag: **Exercise 2. Answer Question 2 in Lab4_Quiz.**

The next step in this lab is to replace this line of code with actual
code that will do the job.

```
System.out.println("We need to send back response headers now.");

```
Next, be sure to remove the code that is only necessary for the initial set up. See the code and read the comments.

The HTTP response headers should include an HTTP code that signals
that all went well, a content-type, and a connection status.

:checkered_flag: **Exercise 3. Answer Question 3 in Lab4_Quiz.**

Next, you will need to replace this line of code with code that will do the job.

```
System.out.println("We need to send back file data now.");

```
:checkered_flag: **Exercise 4. Answer Question 4 in Lab4_Quiz.**

StaringWebServer is working with socket level code.

When working with socket level code, we need to be aware of the following abstractions.
These abstractions are implemented in various languages. For the upcoming exam, you need to be familiar with how these abstractions map on to code.

* Create: Both the client and the server create a socket.
* Bind: The server, but not the client, binds a socket to a local IP address and port.
* Listen: The server, but not the client, listens on a socket.
* Connect: The client, but not the server, assigns a local port and connects to a remote port.
* Accept: The server, but not the client, accepts a connection on a listened to socket.
* Read, Write: Both the client and the server are able to read and write data to and from the socket.
* Close: Both the client and the server close the socket to terminate communication.

Here are some notes (required reading) on HTTP responses.

The minimal response header is simply ```HTTP/1.1 200 OK\n\n```.  The backslash-n indicates a new line. The extra new line means end of headers. A typical server will send additional headers, but you don't need to in this lab.

After the response header and new line, you would follow with the content of the requested file.

In this lab, you only need to handle GET requests (not POST, etc.)

The two html FILES are placed in your IntelliJ project directory to make them easy to find. Other locations will also work.

Return the appropriate HTTP Status Code, (404) if the file is not found, (200) if things went well.

You will visit your simple HTTP server using a browser as the HTTP client.<br>
(You are **not** developing an HTTP client.)

A typical HTTP response header will have the content-length, or "transfer-encoding chunked". We are cheating and just closing the socket. This will indicate to the browser that the response has been completed.

:checkered_flag: **Exercise 5. Answer Question 5 in Lab4_Quiz.**
