### Part 1: Problem-Solution Mapping Table

| Problem (from 1974 paper OR modern problem) | Solution Proposed by Paper OR Why Not Addressed | How We See This Today |
|---------------------------------------------|------------------------------------------------|----------------------|
|Different networks had different packet sizes | Since a packet can be too large for the network following the paper suggests gateweay fragmentation as opposed to forcing every network to use the same packet size | The internet knows what device or network it needs to reach when something is sent over from comptuer to computer like a file or link |
| Packets arriving in the wrong order | The paper addresses how packets were given a sequence number so that the reciever can put them back into the correct order |Parts of a message, like a file, link or even text could take different routes across the internet but it is eventually put back in correct order when the message is sent|
| Data could be corrupt on its way across multiple networks | A set check was made so that the recieving network can verify weather the data arrived correctly | When game files become corrupt the system notifies your computer to initate another installation instead of proceeding with the curropted data|
| Flow Control | The paper address how the reciever networks tells the sender network how much data can be accepted | my phone or computer cannot handle incoming data quickly enough, the sender can slow down instead of flooding it with more information.|
| Network Packet Usage | The paper addresses keeping track of packets between networks for payment or accounting but it doesnt create a complete system for doing it | Internet providers and other companys communicating about the usage of a users incoming traffic|
| Security | The paper only addresses getting different networks to communicate reliably not safely. There is a bit of noticing from the authors about encryption but they do not take into account the reading of data by unwanted systems | HTTPS nowadays encrypts data accross different networks so information cannot be read during travel. Passwords are also a plus!|

## Part 2: AI-Assisted Protocol Investigation

#### A. Investigation Overview
- I chose row 5: Flow control
- How  my phone or my computer actually handle request from other networks to let them know whether they can handle x amount of data. The communication between my system and the network relies on TCP recieving chunks and delivering in a digestable way for the system

#### B. Key Questions I Asked

1. How is the actual "max handling" of a network even decided?
2. What if the recieving computer can never keep up at any moment
3. What happens in TCP when the receiver advertises a zero receive window, and how does the sender know when it can start sending again? 
4. Does the TCP ever stop sending request/stop asking the recieving system if its ready?
5. So to wrap everything together my device uses tcp to as a reciever from another system with information on its current data buffer space. Then if there is no buffer space a series of yes and no's are given from my system to the tcp to decide weather its ready for the data. This is slowly incremented until all the information is given?

#### C. Connection to 1974 Paper (1-2 paragraphs)

To connect this example back to what Certf-Kahn proposed the biggest idea was recieving windows. Having a window that can increase or decrease depedning on the avalible resources are something modern TCP follows even with a more complicated internet. One important change is that TCP today doesnt worry ONLY about overwhelming the device, it worries about overwhelming the network itself as well. Today there are now two limits to TCP; the original reciving window and what they call the Congestion window which is essentially just how much the TCP believes a network path can safely handle. In relation to the second limit of TCP, Cerf-Kahn didnt fully take into account internet congestion, it seems like they spend alot of time focusing on buffer avaliblity but didnt realize as the internet expanding millions of computers had to safely handle enough data to not overload routers and networks links.

#### D. Surprise Finding (4-5 sentences)

How I thought of the recieving window was link a maximum handling amount that a device can take but this was actually wrong. I found it what matters is more of the CURRENT buffer space for incoming data, the AI gave me an analogy of a computer with 100 KB and showed how if it cannot process the fully information then the TCP waits until the computer communicates it can handle more more KB. I also thought there was a terminal state with communication between the TCP and the device, the TCP never STOPS asking the sender if the information is ready if there is a response from the sender. So for example if the sending says "I cant handle anymore information", the TCP would ask "ready now?" and until the sender stops replying the TCP will go indefinetly.

#### E. AI Conversation Documentation
    ME: | Flow Control | The paper address how the reciever networks tells the sender network how much data can be accepted | my phone or computer cannot handle incoming data quickly enough, the sender can slow down instead of flooding it with more information.|

    How is the actual "max handling" of a network even decided?

    ----------------

    AI: The key thing is that the receiver usually does not calculate one permanent “maximum handling speed” for the whole network.
    For flow control, what matters is more like:
    “How much free space do I have right now for incoming data?”

    Suppose your computer has a receive buffer with space for 100 KB of incoming data.
    - The sender sends 60 KB.
    - Your computer has not processed all of it yet.
    - Now only 40 KB of buffer space is free.
    - Your computer tells the sender, essentially: “You may send me 40 KB more.”
    - Your application processes 30 KB.
    - Now 70 KB is free.
    - The receiver can advertise a larger window again.

    The receiver's “maximum” is not usually a fixed number. It is dynamically based on how much data the receiver can currently store and process, while the sender separately tries to estimate how much traffic the network path can carry.

    ----------

    ME: What if the recieving computer can never keep up at any moment 

    ----------

    AI: If the receiving computer can never process incoming data fast enough, then flow control eventually pushes the sender toward sending very little—or temporarily nothing at all.
    In modern TCP, the extreme version of this is a zero window:
    - The receiver says, in effect, “I currently have 0 bytes of space available.”
    - The sender stops sending normal data.
    - The sender periodically checks whether the receiver has freed any space.
    - If the receiving application eventually processes some data, the receiver advertises a larger window and transmission resumes.
    - If the receiver remains stuck for a very long time, the connection may eventually fail because of timeouts or application-level limits.

    ------------

    Me: What happens in TCP when the receiver advertises a zero receive window, and how does the sender know when it can start sending again?

    ------------

    AI: Here’s what happens in modern TCP:
    1. The receiver's buffer fills up. Imagine your computer has a 100 KB receive buffer. The application has not processed the data quickly enough, so all 100 KB are occupied.

    2. The receiver sends an ACK with Window = 0. This tells the sender, “I received everything up to this point, but I currently have zero additional space.” The sender then stops sending new normal data. Modern TCP specifically says the sender should not send new data when the advertised window has shrunk to zero.

    3. Eventually the receiving application processes some data. Maybe the application removes 40 KB from the buffer. Now the receiver has 40 KB of free space.

    4. The receiver can send a window update. It might now advertise Window = 40 KB, effectively saying, “I have room again.”

    TCP solves that with zero-window probing. While the window is zero, the sender periodically sends a tiny probe. The receiver must respond with an ACK containing its current window size.

    The probe is important because it gives the sender a way to ask:
    “Are you still full?”

    TCP doesn't guess when the receiver is ready again. The receiver advertises how much space it has, and while that value is zero, the sender periodically probes the receiver to obtain an updated value.

    ------------

    Me: Does the TCP ever stop sending request/stop asking the recieving system if its ready?

    ------------
    
    AI: At the TCP protocol level, not necessarily. If the receiver keeps advertising a zero window but continues replying to the sender’s probes, TCP is allowed to keep the connection open indefinitely. The sender keeps probing, but it backs off so it asks less and less frequently rather than constantly hammering the receiver. RFC 9293 says the first zero-window probe should occur after a retransmission-timeout interval, and the interval between later probes should increase exponentially

    -----------

    Me: So to wrap everything together my device uses tcp to as a reciever from another system with information on its current data buffer space. Then if there is no buffer space a series of yes and no's are given from my system to the tcp to decide weather its ready for the data. This is slowly incremented until all the information is given?

    ------------

    AI: Yes — you have the main idea. I would just change two parts: it is not really a series of yes/no responses, and the amount does not necessarily increase slowly.

    A more accurate way to put everything together is:
    My device uses TCP to tell the sending system how much free space is currently available in its receive buffer. The sender uses that information to control how much data it sends so it does not overwhelm my device. If the receive buffer becomes completely full, my device can advertise a window of zero, causing the sender to stop sending normal data. As my device processes the existing data and frees buffer space, it advertises a larger window, allowing the sender to continue. This process repeats until all of the data has been transferred.

## Part 3: Reflections

Personally I wanted to learn more about the security and authentication. I guess in the early phases of the internet they werent really thinking of the "bad" it can do only the potential advances in how we think about networking. I use AI to study frequently and I always tell others ask the AI questions like you have NO idea what youre talking about, the AI isnt demening at all so you dont have to worry about it judging you and specifically topics like this where I have no idea what the paper was talking about. Asking it to first summarize the paper then finish the assingments based off of that summerization is way more effienct then reading something you do not understand. Not really, I always thought the internet was a wide web of complex features put together, I guess this process updated my understanding of how smart our founding fathers were in inventing something so great!