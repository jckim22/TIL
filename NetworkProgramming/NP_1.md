# SingleFileHTTPServer

해당 서버에 접속하면 고정된 파일 하나를 던져주는 서버이다.

```java
public class SingleFileHTTPServer {
    private static final Logger logger = Logger.getLogger("SingleFileHTTPServer");
    private final byte[] content;
    private final byte[] header;
    private final int port;
    private final String encoding;

    public SingleFileHTTPServer(String data, String encoding, String mimeType, int port) throws UnsupportedEncodingException {
        this(data.getBytes(encoding), encoding, mimeType, port);
    }

    public SingleFileHTTPServer(
            byte[] data, String encoding, String mimeType, int port) {
        this.content = data;
        this.port = port;
        this.encoding = encoding;
        String header = "HTTP/1.0 200 OK\r\n"
                + "Server: OneFile 2.0\r\n"
                + "Content-length: " + this.content.length + "\r\n"
                + "Content-type: " + mimeType + "; charset=" + encoding + "\r\n\r\n";
        this.header = header.getBytes(Charset.forName("US-ASCII"));
    }

    public void start() {
        ExecutorService pool = Executors.newFixedThreadPool(100);
        try (ServerSocket server = new ServerSocket(this.port)) {
            logger.info("Accepting connections on port " + server.getLocalPort());
            logger.info("Data to be sent:");
            logger.info(new String(this.content, encoding));
            while (true) {
                try {
                    Socket connection = server.accept();
                    pool.submit(new HTTPHandler(connection));
                } catch (IOException ex) {
                    logger.log(Level.WARNING, "Exception accepting connection", ex);
                } catch (RuntimeException ex) {
                    logger.log(Level.SEVERE, "Unexpected error", ex);
                }
            }
        } catch (IOException ex) {
            logger.log(Level.SEVERE, "Could not start server", ex);
        }
    }

    private class HTTPHandler implements Callable<Void> {
        private final Socket connection;

        HTTPHandler(Socket connection) {
            this.connection = connection;
        }

        @Override
        public Void call() throws IOException {
            try {
                OutputStream out = new BufferedOutputStream(connection.getOutputStream()
                );
                InputStream in = new BufferedInputStream(
                        connection.getInputStream()
                );
                // read the first line only; that's all we need
                StringBuilder request = new StringBuilder(80);
                while (true) {
                    int c = in.read();
                    if (c == '\r' || c == '\n' || c == -1) break;
                    request.append((char) c);
                }
                // If this is HTTP/1.0 or later send a MIME header
                if (request.toString().indexOf("HTTP/") != -1) {
                    out.write(header);
                }
                out.write(content);
                out.flush();
            } catch (IOException ex) {
                logger.log(Level.WARNING, "Error writing to client", ex);
            } finally {
                connection.close();
            }
            return null;
        }
    }

    public static void main(String[] args) {
        // set the port to listen on
        int port=12345;
//        try {
//            port = 12345;
//            if (port < 1 || port > 65535)
//                port = 80;
//        } catch (RuntimeException ex) {
//            port = 80;
//        }

        String encoding = "UTF-8";
//        if (args.length > 2) encoding = args[2];
        try {
            String filename = "/Users/jckim2/Desktop/Network_Programming/return.html";
//            Path path = Paths.get(args[0]);
            Path path = Paths.get(filename);
            byte[] data = Files.readAllBytes(path);
            String contentType = URLConnection.getFileNameMap().getContentTypeFor(filename);
            SingleFileHTTPServer server = new SingleFileHTTPServer(data, encoding,
                    contentType, port);
            server.start();
        } catch (ArrayIndexOutOfBoundsException ex) {
            System.out.println(
                    "Usage: java SingleFileHTTPServer filename port encoding");
        } catch (IOException ex) {
            logger.severe(ex.getMessage());
        }
    }
}

```

port를 12345로 설정하고 서버를 연다.
HTTPHandler는 Callable로 구현되었다.

스레드 풀을 생성한뒤 accept로 클라이언트의 요청을 기다리다가 요청이 들어오면 submit으로 풀에 던져주며 실행한다.

![업로드중..](blob:https://velog.io/cdb268e8-b2b5-4296-8de1-a67771097a1b)

접속해보면 위처럼 내가 던진 html 파일이 출력된다.

# RedirectorHTTPServer

이번에는 서버에 접속하면 HTTP Response의 Location으로 지정한 사이트로 리다이렉트 되는 HTTP서버를 만들었다.

```java
public class Redirector {
    private static final Logger logger = Logger.getLogger("Redirector");
    private final int port;
    private final String newSite;

    public Redirector(String newSite, int port) {
        this.port = port;
        this.newSite = newSite;
    }

    public void start() {
        try (ServerSocket server = new ServerSocket(port)) {
            logger.info("Redirecting connections on port "
                    + server.getLocalPort() + " to " + newSite);
            while (true) {
                try {
                    Socket s = server.accept();
                    Thread t = new RedirectThread(s);
                    t.start();
                } catch (IOException ex) {
                    logger.warning("Exception accepting connection");
                } catch (RuntimeException ex) {
                    logger.log(Level.SEVERE, "Unexpected error", ex);
                }
            }
        } catch (BindException ex) {
            logger.log(Level.SEVERE, "Could not start server.", ex);
        } catch (IOException ex) {
            logger.log(Level.SEVERE, "Error opening server socket", ex);
        }
    }

    private class RedirectThread extends Thread {
        private final Socket connection;

        RedirectThread(Socket s) {
            this.connection = s;
        }

        public void run() {
            try {
                Writer out = new BufferedWriter(
                        new OutputStreamWriter(
                                connection.getOutputStream(), "US-ASCII"
                        )
                );
                Reader in = new InputStreamReader(
                        new BufferedInputStream(connection.getInputStream()
                        ));
                // read the first line only; that's all we need
                StringBuilder request = new StringBuilder(80);
                while (true) {
                    int c = in.read();
                    if (c == '\r' || c == '\n' || c == -1) break;
                    request.append((char) c);
                }
                String get = request.toString();
                String[] pieces = get.split("\\w*");
                String theFile = pieces[1];
                // If this is HTTP/1.0 or later send a MIME header
                if (get.indexOf("HTTP") != -1) {
                    out.write("HTTP/1.0 302 FOUND\r\n");
                    Date now = new Date();
                    out.write("Date: " + now + "\r\n");
                    out.write("Server: Redirector 1.1\r\n");
                    out.write("Location: " + newSite + theFile + "\r\n");
                    out.write("Content-type: text/html\r\n\r\n");
                    out.flush();
                }
// Not all browsers support redirection so we need to
// produce HTML that says where the document has moved to. out.write("<HTML><HEAD><TITLE>Document moved</TITLE></HEAD>\r\n"); out.write("<BODY><H1>Document moved</H1>\r\n");
                out.write("The document " + theFile
                        + " has moved to\r\n<A HREF=\"" + newSite + theFile + "\">"
                        + newSite + theFile
                        + "</A>.\r\n Please update your bookmarks<P>");
                out.write("</BODY></HTML>\r\n");
                out.flush();
                logger.log(Level.INFO,
                        "Redirected " + connection.getRemoteSocketAddress());
            } catch (IOException ex) {
                logger.log(Level.WARNING,
                        "Error talking to " + connection.getRemoteSocketAddress(), ex);
            } finally {
                try {
                    connection.close();
                } catch (IOException ex) {
                }
            }
        }
    }

    public static void main(String[] args) {
        int thePort=12347;
        String theSite;

        try {
            theSite = "https://www.naver.com";
// trim trailing slash
            if (theSite.endsWith("/")) {
                theSite = theSite.substring(0, theSite.length() - 1);
            }
        } catch (RuntimeException ex) {
            System.out.println(
                    "Usage: java Redirector http://www.newsite.com/ port");
            return;
        }
//        try {
//            thePort = Integer.parseInt(args[1]);
//        } catch (RuntimeException ex) {
//            thePort = 80;
//        }
        Redirector redirector = new Redirector(theSite, thePort);
        redirector.start();
    }
}
```

SingleFileHttpServer와 비슷하지만 out.write로 Response에 지정한 네이버로 Redirect 되게 만들었다.

# JHTTP

실제 HTTP서버와 비슷하게 동작하는 웹 서버를 만들었다.

```java
public class JHTTP {
    private static final Logger logger = Logger.getLogger(JHTTP.class.getCanonicalName());
    private static final int NUM_THREADS = 50;
    private static final String INDEX_FILE = "index.html";
    private final File rootDirectory;
    private final int port;

    public JHTTP(File rootDirectory, int port) throws IOException {
        if (!rootDirectory.isDirectory()) {
            throw new IOException(rootDirectory
                    + " does not exist as a directory");
        }
        this.rootDirectory = rootDirectory;
        this.port = port;
    }

    public void start() throws IOException {
        ExecutorService pool = Executors.newFixedThreadPool(NUM_THREADS);
        try (ServerSocket server = new ServerSocket(port)) {
            logger.info("Accepting connections on port " + server.getLocalPort());
            logger.info("Document Root: " + rootDirectory);
            while (true) {
                try {
                    Socket request = server.accept();
                    Runnable r = new RequestProcessor(
                            rootDirectory, INDEX_FILE, request);
                    pool.submit(r);
                } catch (IOException ex) {
                    logger.log(Level.WARNING, "Error accepting connection", ex);
                }
            }
        }
    }

    public static void main(String[] args) {
        // get the Document root
        File docroot = new File("/Users/jckim2/Desktop/Network_Programming/src/HTTPServer");
//        try {
//            docroot = new File(args[0]);
//        } catch (ArrayIndexOutOfBoundsException ex) {
//            System.out.println("Usage: java JHTTP docroot port");
//            return;
//        }
        // set the port to listen on
        int port = 20000;
//        try {
//            port = 20000;
//            if (port < 0 || port > 65535) port = 80;
//        } catch (RuntimeException ex) {
//            port = 80;
//        }
        try {
            JHTTP webserver = new JHTTP(docroot, port);
            webserver.start();
        } catch (IOException ex) {
            logger.log(Level.SEVERE, "Server could not start", ex);
        }
    }
}
```

## RequestProcessor

```java
public class RequestProcessor implements Runnable {
    private final static Logger logger = Logger.getLogger(RequestProcessor.class.getCanonicalName());
    private File rootDirectory;
    private String indexFileName = "index.html";
    private Socket connection;

    public RequestProcessor(File rootDirectory, String indexFileName, Socket connection) {
        if (rootDirectory.isFile()) {
            throw new IllegalArgumentException(
                    "rootDirectory must be a directory, not a file");
        }
        try {
            rootDirectory = rootDirectory.getCanonicalFile();
        } catch (IOException ex) {
        }
        this.rootDirectory = rootDirectory;
        if (indexFileName != null) this.indexFileName = indexFileName;
        this.connection = connection;
    }

    @Override
    public void run() {
// for security checks
        String root = rootDirectory.getPath();
        try {
            OutputStream raw = new BufferedOutputStream(connection.getOutputStream()
            );
            Writer out = new OutputStreamWriter(raw);
            Reader in = new InputStreamReader(new BufferedInputStream(
                    connection.getInputStream()
            ), "US-ASCII"
            );
            StringBuilder requestLine = new StringBuilder();
            while (true) {
                int c = in.read();
                if (c == '\r' || c == '\n') break;
                requestLine.append((char) c);
            }
            String get = requestLine.toString();
            logger.info(connection.getRemoteSocketAddress() + " " + get);
            String[] tokens = get.split("\\s+");
            String method = tokens[0];
            String version = "";
            if (method.equals("GET")) {
                String fileName = tokens[1];
                if (fileName.endsWith("/")) fileName += indexFileName;
                String contentType =
                        URLConnection.getFileNameMap().getContentTypeFor(fileName);
                if (tokens.length > 2) {
                    version = tokens[2];
                }
                File theFile = new File(rootDirectory, fileName.substring(1, fileName.length()));
                if (theFile.canRead() && theFile.getCanonicalPath().startsWith(root)) {
                    byte[] theData = Files.readAllBytes(theFile.toPath());
                    if (version.startsWith("HTTP/")) { // send a MIME header
                        sendHeader(out, "HTTP/1.0 200 OK", contentType, theData.length);
                    }
// send the file; it may be an image or other binary data // so use the underlying output stream
// instead of the writer
                    raw.write(theData);
                    raw.flush();
                } else { // can't find the file
                    String body = new StringBuilder("<HTML>\r\n").append("<HEAD><TITLE>File Not Found</TITLE>\r\n").append("</HEAD>\r\n")
                            .append("<BODY>")
                            .append("<H1>HTTP Error 404: File Not Found</H1>\r\n")
                            .append("</BODY></HTML>\r\n").toString();
                    if (version.startsWith("HTTP/")) { // send a MIME header
                        sendHeader(out, "HTTP/1.0 404 File Not Found",
                                "text/html; charset=utf-8", body.length());
                    }
                    out.write(body);
                    out.flush();
                }
            } else { // method does not equal "GET"
                String body = new StringBuilder("<HTML>\r\n").append("<HEAD><TITLE>Not Implemented</TITLE>\r\n").append("</HEAD>\r\n")
                        .append("<BODY>")
                        .append("<H1>HTTP Error 501: Not Implemented</H1>\r\n")
                        .append("</BODY></HTML>\r\n").toString();
                if (version.startsWith("HTTP/")) { // send a MIME header
                    sendHeader(out, "HTTP/1.0 501 Not Implemented",
                            "text/html; charset=utf-8", body.length());
                }
                out.write(body);
                out.flush();
            }
        } catch (IOException ex) {
            logger.log(Level.WARNING,
                    "Error talking to " + connection.getRemoteSocketAddress(), ex);
        } finally {
            try {
                connection.close();
            } catch (IOException ex) {
            }
        }
    }

    private void sendHeader(Writer out, String responseCode, String contentType, int length)
            throws IOException {
        out.write(responseCode + "\r\n");
        Date now = new Date();
        out.write("Date: " + now + "\r\n");
        out.write("Server: JHTTP 2.0\r\n");
        out.write("Content-length: " + length + "\r\n");
        out.write("Content-type: " + contentType + "\r\n\r\n");
        out.flush();
    }
}
```

handler는 RequsetProcessor에서 runnable을 구현한다.

GET 요청이아니면 구현 되지 않았다는 응답을 보낸다.

루트 디렉토리를 정하고 특정 파일이 정해지지않았다면 index.html을 리턴하고 특정 파일을 고르면 그 파일을 리턴한다.

### RootDirectory
![업로드중..](blob:https://velog.io/3165fb95-fa29-45ff-811d-3e52eb987c1d)

루트 디렉토리를 생성하고 그 안에 html파일을 생성한다.

### index.html
![업로드중..](blob:https://velog.io/c59730f6-9583-4f94-9a14-fbe9bc6a42a1)
### hi.html
![업로드중..](blob:https://velog.io/c1c7ae55-e27f-40d8-97ef-0de9ca195483)

그 후 설정한 포트를 통해 서버에 접속하면 아래처럼 html파일이 리턴된다.


![업로드중..](blob:https://velog.io/a619bff7-3c3e-4543-80d8-22c2405e46c2)
![업로드중..](blob:https://velog.io/b502dabd-94e8-4def-a0c4-f621f3f3e436)
