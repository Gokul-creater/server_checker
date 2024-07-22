Nice script. Here are a few points that can be improved.

You could use a dictionnary to handle the conversion from priority to x-priority.

PRIORITY_TO_XPRIORITY = {
    LOW: '5',  # ThunderBird "Lowest", works with Exchange.
    NORMAL: 'Normal',  # Plain old "Normal". Works with Exchange.
    HIGH: '1',  # ThunderBird "Highest", works with Exchange.
}
...
msg['X-Priority'] = PRIORITY_TO_XPRIORITY[priority]
You don't need the continue in your for loop after the socket.timeout.

You probably should fail in a more explicit way when given an invalid connection mechanism.

More generally, having the exit() function called from anywhere makes things a bit hard to track.

You do not need global variable to propagate the list of servers up/down : a list populated in the main and fed to send_server_status_report should do the trick.

You could define a function to check a server status.

Then you could call it to know if the server is up without bothering about the way it is done.

You main function simply becomes :

def server_is_up(server):
    srv, mechanism, port = server
    try:
        if mechanism == 'plain':
            # Use a plain text connector for this.
            print "%s  Using Plain for [%s]..." % (current_timestamp(), srv)
            socket.create_connection(("%s.layerbnc.org" % srv, port), timeout=10)
        elif mechanism == 'ssl':
            # We're going to use an SSL connector for this.
            print "%s  Using SSL for [%s]..." % (current_timestamp(), srv)
            ssl.wrap_socket(socket.create_connection(("%s" % srv, port), timeout=10))
        else:
            print "%s  Invalid mechanism defined for [%s], skipping..." % (current_timestamp(), srv)
            raise ValueError("Invalid mechanism")
        print "%s  %s: UP" % (current_timestamp(), srv)
        return True
    except socket.timeout:
        print "%s  %s: DOWN" % (current_timestamp(), srv)
        return False
    except Exception as err:
        print "An error occurred: %s" % (err.__str__())
        exit()


def main():
    srv_up = []
    svr_down = []
    for server in sorted(SERVER_LIST):
        (srv_up if server_is_up(server) else svr_down).append(server[0])
    send_server_status_report(svr_up, svr_down)  # Create email to send the status notices.

    exit()  # Exit when done
I do not think you need to define smtp being None before the try/catch. Scopes in Python do not quite work like in other languages.

I know you do not want to talk about this but your current_timestamp function could be removed if you were to use a proper logging function. If you really do not want to do this, you could at least define a log function doing exactly what you are doing but calling the current_timestamp function automatically so that you do not have to do this every where in your code.
