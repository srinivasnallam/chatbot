# chatbot
#seperately run server and client file in python idle 
#First run server and then client.Copy the ip generated at server and paste in the client output and u r ready to go :)

Server.py:


from __future__ import print_function
import socket
import sys
import time
from _thread import *
import os

import re
import random

from nltk.chat.util import Chat, reflections

message = ""

class Chat(object):
    def __init__(self, pairs, reflections={}):
        """
        Initialize the chatbot.  Pairs is a list of patterns and responses.  Each
        pattern is a regular expression matching the user's statement or question,
        e.g. r'I like (.*)'.  For each such pattern a list of possible responses
        is given, e.g. ['Why do you like %1', 'Did you ever dislike %1'].  Material
        which is matched by parenthesized sections of the patterns (e.g. .*) is mapped to
        the numbered positions in the responses, e.g. %1.

        :type pairs: list of tuple
        :param pairs: The patterns and responses
        :type reflections: dict
        :param reflections: A mapping between first and second person expressions
        :rtype: None
        """

        self._pairs = [(re.compile(x, re.IGNORECASE), y) for (x, y) in pairs]
        self._reflections = reflections
        self._regex = self._compile_reflections()

    def _compile_reflections(self):
        sorted_refl = sorted(self._reflections.keys(), key=len, reverse=True)
        return re.compile(
            r"\b({0})\b".format("|".join(map(re.escape, sorted_refl))), re.IGNORECASE
        )

    def _substitute(self, str):
        """
        Substitute words in the string, according to the specified reflections,
        e.g. "I'm" -> "you are"

        :type str: str
        :param str: The string to be mapped
        :rtype: str
        """

        return self._regex.sub(
            lambda mo: self._reflections[mo.string[mo.start() : mo.end()]], str.lower()
        )

    def _wildcards(self, response, match):
        pos = response.find('%')
        while pos >= 0:
            num = int(response[pos + 1 : pos + 2])
            response = (
                response[:pos]
                + self._substitute(match.group(num))
                + response[pos + 2 :]
            )
            pos = response.find('%')
        return response

    def respond(self, str):
        """
        Generate a response to the user input.

        :type str: str
        :param str: The string to be mapped
        :rtype: str
        """

        # check each pattern
        for (pattern, response) in self._pairs:
            match = pattern.match(str)

            # did the pattern match?
            if match:
                resp = random.choice(response)# pick a random response
                
                resp = self._wildcards(resp, match)  # process wildcards

                # fix munged punctuation at the end
                if resp[-2:] == '?.':
                    resp = resp[:-2] + '.'
                if resp[-2:] == '??':
                    resp = resp[:-2] + '?'
                return resp


    # Hold a conversation with a chatbot
    def converse(self,inc):
        user_input = ""
        try:
            user_input = inc
        except EOFError:
            print(user_input)
        if user_input:
            while user_input[-1] in "!.":
                user_input = user_input[:-1]
            return self.respond(user_input)


pairs = [
    [
        r"hi im (.*)",
        ["Hello %1 How are you today ?",]
    ],
     [
        r"what is your name ?",
        ["My name is Chatty and I'm a chatbot :)",]
    ],
    [
        r"how are you ?",
        ["I'm doing good\nHow about You ?",]
    ],
    [
        r"sorry (.*)",
        ["Its alright","Its OK, never mind",]
    ],
    [
        r"i'm (.*) doing good",
        ["Nice to hear that","Alright :)",]
    ],
    [
        r"hi|hey|hello",
        ["Hello", "Hey there",]
    ],
    [
        r"(.*) age?",
        ["I'm a computer program dude\nSeriously you are asking me this?",]
        
    ],
    [
        r"what (.*) want ?",
        ["Make me an offer I can't refuse",]
        
    ],
    [
        r"(.*) created ?",
        ["Sahas created me using Python's NLTK library ","top secret :)",]
    ],
    [
        r"(.*) (location|city) ?",
        ['Hyderabad, Telangana',]
    ],
    [
        r"how is weather in (.*)?",
        ["Weather in %1 is awesome like always","Too hot man here in %1","Too cold man here in %1","Never even heard about %1"]
    ],
    [
        r"i work in (.*)?",
        ["%1 is an Amazing company, I have heard about it. But they are in huge loss these days.",]
    ],
    [
        r"i love (.*)",
        ["I love you too :)",]
    ],
    [
        r"(.*)raining in (.*)",
        ["No rain since last week here in %2","Damn its raining too much here in %2"]
    ],
    [
        r"how (.*) health(.*)",
        ["I'm a computer program, so I'm always healthy ",]
    ],
    [
        r"(.*) (sports|game) ?",
        ["I'm a very big fan of Football",]
    ],
    [
        r"who (.*) sportsperson ?",
        ["Messy","Ronaldo","Roony"]
],
    [
        r"who (.*) (moviestar|actor)?",
        ["Brad Pitt"]
],
    [
        r"quit",
        ["BBye take care. See you soon :) ","It was nice talking to you. See you soon :)"]
        
],
    [
        r"(.*)price of (.*) product",
        ["Product name - %2 \n Price - 1500rs","Product name - %2 \n Price - 2500rs","Product name - %2 \n Price - 1700rs"]
        
],
]
#socket Creation
s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

host = socket.gethostname()

ip = socket.gethostbyname(host)

print("Server will start on host:",ip)

port = 8080

s.bind((ip,port))

print("Server done binding host and port")

print("Server is waiting for incomming connections .......")


            
def clientthread(conn):
    
    while 1:

        
        
        incomming_message = conn.recv(1024)
        
        incomming_message = incomming_message.decode()

        def chatty():
            chat = Chat(pairs, reflections)
            
            message = chat.converse(incomming_message)

            message = str(message)

            if (message != 'None'):
            
                message = message.encode()

                conn.send(message)
            
            else :

                message1 = "Your Request has been forwarded to Admin team Please wait while we get back..... :) "

                message1 = message1.encode()

                conn.send(message1)

            

        chatty()

        
        
        


    
while 1:
    
    s.listen(1)
    
    conn, addr = s.accept()
    
    print(addr,"Connected to the server and is now online.....")
    
    start_new_thread(clientthread, (conn,))



Client.py:

import socket
import sys
import time
from _thread import *
import os

#socket Creation
s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

#Taking the ip address of server to establish connection
ip = input(str("Please enter the host IP:"))

port = 8080

#connecting to server
s.connect((ip,port))

print("Connected to the server.......")

while 1 :

    message = input(str(">>"))
    
    message = message.encode()
    
    s.send(message)
    
    incomming_message = s.recv(1024)
    
    incomming_message = incomming_message.decode()
    
    print("Server:",incomming_message)
    
    
     

