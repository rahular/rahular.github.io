---
id: 183
title: Stockfish Port for Java
author: rahular
layout: post
excerpt: A hacky java port of the famous Stockfish. Should be able to perform relatively simple tasks well. Might even be able to play a standard time controlled game of chess
guid: http://rahular.com/?p=183
permalink: /stockfish-port-for-java/
categories:
  - Misc
---
The usual problem with applications written in C/C++ is that we cannot directly use it&#8217;s functionality in the newer languages like Java, Python, etc. without some sort of glue such as JNI.

I faced this problem in one of my course projects. I had use use some of the functionality of the amazing Stockfish chess engine in my research. Being a Java fanatic, writing a thin client to integrate things seemed cumbersome and boring.

So I wrote a simple hack and docked pipes into Stockfish&#8217;s IO and routed everything into a Java class. Since then I have improved it quite a bit so that anyone interested in building something to do with Chess can use it in their projects.

Here is a brief overview of what it can do.

  1. Start the engine
  2. Stop the engine
  3. Get list of valid moves
  4. Get best move
  5. Send UCI commands to engine
  6. Get raw dump from engine
  7. Pretty print the board state
  8. Get evaluation score

```java
Stockfish client = new Stockfish();
String FEN = "8/6pk/8/1R5p/3K3P/8/6r1/8 b - - 0 42";

// initialize and connect to engine
if (client.startEngine()) {
	System.out.println("Engine has started..");
} else {
	System.out.println("Oops! Something went wrong..");
}

// send commands manually
client.sendCommand("uci");

// receive output dump
System.out.println(client.getOutput(0));

// get the best move for a position with a given think time
System.out.println("Best move : " + client.getBestMove(FEN, 100));

// get all the legal moves from a given position
System.out.println("Legal moves : " + client.getLegalMoves(FEN));

// draw board from a given position
System.out.println("Board state :");
client.drawBoard(FEN);

// get the evaluation score of current position
System.out.println("Eval score : " + client.getEvalScore(FEN, 2000));

// stop the engine
System.out.println("Stopping engine..");
client.stopEngine();
```

The code is well commented and extremely simple which can be forked from <a title="GitHub" href="https://github.com/rahular/chess-misc" target="_blank">GitHub</a>. Stockfish&#8217;s UCI can be directly invoked as shown in the code snippet. But be mindful of the computation time before getting the output dump. You have worry about this timing constraint only if you prefer to manually enter commands using sendCommand and get the raw dump using getOutput. Otherwise, everything is handled for you internally.

Enjoy!
