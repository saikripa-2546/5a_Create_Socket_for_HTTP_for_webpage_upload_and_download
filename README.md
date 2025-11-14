# 5a_Create_Socket_for_HTTP_for_webpage_upload_and_download
## AIM :
To write a PYTHON program for socket for HTTP for web page upload and download
## Algorithm

1.Start the program.
<BR>
2.Get the frame size from the user
<BR>
3.To create the frame based on the user request.
<BR>
4.To send frames to server from the client side.
<BR>
5.If your frames reach the server it will send ACK signal to client otherwise it will send NACK signal to client.
<BR>
6.Stop the program
<BR>
## Program 
```
import socket
import threading
import time


def send_request(host, port, request_bytes):
    with socket.create_connection((host, port), timeout=5) as s:
        s.sendall(request_bytes)
        chunks = []
        while True:
            data = s.recv(4096)
            if not data:
                break
            chunks.append(data)
    return b"".join(chunks)


def upload_file(host, port, filename):
    with open(filename, "rb") as file:
        file_data = file.read()

    headers = (
        f"POST /upload/{filename} HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"Content-Length: {len(file_data)}\r\n"
        "Connection: close\r\n"
        "\r\n"
    ).encode("utf-8")

    response = send_request(host, port, headers + file_data)
    return response.decode(errors="ignore")


def download_file(host, port, filename, destination=None):
    request = (
        f"GET /{filename} HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        "Connection: close\r\n"
        "\r\n"
    ).encode("utf-8")

    response = send_request(host, port, request)
    header_body_split = response.split(b"\r\n\r\n", 1)
    if len(header_body_split) != 2:
        raise RuntimeError("Invalid HTTP response received.")

    body = header_body_split[1]
    target_file = destination or filename
    with open(target_file, "wb") as file:
        file.write(body)

    return body


def start_http_server(host, port):
    def handle_client(conn):
        with conn:
            data = b""
            while b"\r\n\r\n" not in data:
                chunk = conn.recv(4096)
                if not chunk:
                    break
                data += chunk

            if not data:
                return

            header_bytes, _, body = data.partition(b"\r\n\r\n")
            header_lines = header_bytes.decode().split("\r\n")
            request_line = header_lines[0]
            method, path, _ = request_line.split()

            headers = {}
            for line in header_lines[1:]:
                if ": " in line:
                    key, value = line.split(": ", 1)
                    headers[key.lower()] = value

            content_length = int(headers.get("content-length", "0"))
            while len(body) < content_length:
                chunk = conn.recv(4096)
                if not chunk:
                    break
                body += chunk

            response = build_response(method, path, body)
            conn.sendall(response)

    def build_response(method, path, body):
        if method == "POST" and path.startswith("/upload/"):
            filename = path[len("/upload/") :]
            with open(filename, "wb") as file:
                file.write(body)
            response_body = b"Upload successful."
            return (
                "HTTP/1.1 200 OK\r\n"
                f"Content-Length: {len(response_body)}\r\n"
                "Content-Type: text/plain\r\n"
                "Connection: close\r\n"
                "\r\n"
            ).encode("utf-8") + response_body

        if method == "GET":
            filename = path.lstrip("/")
            try:
                with open(filename, "rb") as file:
                    file_bytes = file.read()
                return (
                    "HTTP/1.1 200 OK\r\n"
                    f"Content-Length: {len(file_bytes)}\r\n"
                    "Content-Type: application/octet-stream\r\n"
                    "Connection: close\r\n"
                    "\r\n"
                ).encode("utf-8") + file_bytes
            except FileNotFoundError:
                response_body = b"File not found."
                return (
                    "HTTP/1.1 404 Not Found\r\n"
                    f"Content-Length: {len(response_body)}\r\n"
                    "Content-Type: text/plain\r\n"
                    "Connection: close\r\n"
                    "\r\n"
                ).encode("utf-8") + response_body

        response_body = b"Method not allowed."
        return (
            "HTTP/1.1 405 Method Not Allowed\r\n"
            f"Content-Length: {len(response_body)}\r\n"
            "Content-Type: text/plain\r\n"
            "Connection: close\r\n"
            "\r\n"
        ).encode("utf-8") + response_body

    def server_loop():
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
            server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
            server.bind((host, port))
            server.listen()
            while True:
                conn, _ = server.accept()
                threading.Thread(target=handle_client, args=(conn,), daemon=True).start()

    threading.Thread(target=server_loop, daemon=True).start()


if __name__ == "__main__":
    host = "127.0.0.1"
    port = 8080

    start_http_server(host, port)
    time.sleep(0.2)  # Give the server a moment to start

    upload_response = upload_file(host, port, "example.txt")
    print("Upload response:", upload_response)

    downloaded_body = download_file(host, port, "example.txt", "downloaded_example.txt")
    print("File downloaded successfully.")
    print("\nDownloaded HTML content:\n")
    print(downloaded_body.decode(errors="ignore"))
```
## OUTPUT
<img width="1007" height="512" alt="image" src="https://github.com/user-attachments/assets/273b537d-68d8-4478-b71d-d2e4cefce878" />

## Result
Thus the socket for HTTP for web page upload and download created and Executed
