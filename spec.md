General:

A mevent is a pair of strings
1. the port name
2. the payload.

A Leaf Part is a command-line program that contains a single input queue of mevents and a single output queue of mevents. A Part can have multiple named input ports and multiple named output ports. The code within a Leaf Part may be written in any programming language, for example, Python, Javascript, etc.

The default input port is named "" and the default output port is named "".

A Part has three entry points
1. enqueue - which takes a single mevent and appends it to the end of its own input queue.
2. react - if there are any mevents in the input queue, the Part removes the first mevent from its input queue and reacts to it. Reactions are customized for each Part based on the port name. The reactions for each Part are specified below
3. fetch - returns the queue of output mevents that were created in reaction to a single input mevent.

A Leaf Part is essentially a case statement based on the port name of the mevent being reacted to with further code inside each case to handle the payload.

A Container Part contains a list (or array) of children Parts and a table of connections between Parts. Contained Parts can be Leaf Parts or other Container Parts, i.e. Container Parts are defined recursively, whereas Leaf Parts are not defined recursively and contain actual action code written in any programming language written in any paraidmg. The list of Connnections is in JSON format which will be described further below.

A Connection is a triple
1. direction
2. source Part and source port
3. receiver Part and receiver port.

Source and Receiver Parts refer to parts on the list of children Parts owned by the Container.

Direction is one of
1. down
2. across
3. up
4. through.

A `down` connection sends a copy of the reacted-to mevent to one or more children Parts. The mevent is cloned and rewritten for each receiver Part so that the new mevent.port name is changed to be the same as that of of the connection.receiver.port

An `across` connection sends a copy of the reacted-to mevent to one or more sibling Parts within the same Container. The receivers must all be in the child list of the same Container. The mevent is cloned and rewritten for each receiver Part so that the new mevent.port name is changed to be the same as that of of the connection.receiver.port. A Part is allowed to send mevents to its own inputs. In such a case, the mevent is appended to the input queue, just like any mevent from elsewhere. A FIFO queuing protocol is used, not the familiar LIFO stack-based protocol that is used for recursive functions.

An `up` connection sends a copy of the reacted-to mevent from a child Part to the output queue of the Container. The mevent is copied and the port name of the new copy is set to be the name of the destination output port of the Container.

A `through` connection sends a copy of the reacted-to mevent sends a copy of the reacted-to from the Container's input queue to the output queue of the same Container. The mevent is copied and the port name of the new copy is set to be the name of the destination output port of the Container.

All connections are processed atomically for each input mevent to a Container. A Container may not react to another mevent until all of its children have finished reacting to the same mevent. A Container loops through its connections, shepherding mevents, until no children Parts have any mevents on their input queues.

When a Part.react() entry point is called, the Part processes its first mevent (if any) to completion. When it returns control back to the caller, the caller calls the Part.fetch() entry point to fetch the queue of generated mevents. If the called Part is a child, then its Container distributes the output mevents in the usual way, using the Connection table. If the called Part is at the top level, then its outputs are formatted in JSON and sent to stdout.

JSON format of Container Parts:

A Container is a JSON object with NNN fields:
1. "name" - the name of the Container
2. "children" - an array of JSON objects that contains a pair `{ "name" : "...", "id" : nn }` for each child parts. "..." is the name of the template for the child part. nn is a unique numeric id for the instance of that child within the Container. A JSON Container is just a template for a runtime Container. A Container's template is instantiated at load time by recursively instantiating all contained children at load time. A Container's children refer to Part templates. The templates are instantiated as unique Parts with unique ids at load time. A Container may contain more than one Part with the same template, but, at load time, each child Part is uniquely instantiated from the given template.
3. "connections" - an array of JSON objects representing each and every connection between Parts inside a Container ("across") and the Container and its children ("down") and between Parts and their own Container ("up") and from the Container's own inputs to its own outputs ("through"). Fan-out connections - from one output to many receivers - are represented by one 1:1 connection for each connection. Likewise for fan-in, where each 1:1 connection is represented by one complete JSON connection object. Connection objects are described below. Connection objects are analogous to single wires in an electronic circuit.
4. "file" - a string name of the file which was used to generate the Container. This field is informational only. Currently, Containers are represented as drawings in draw.io, hence the filename will refer to a .drawio file at present.

A JSON Connection object contains up to 5 fields:
1. "dir" - 0 for `down`, 1 for `across`, 2 for `up` and 3 for `through`
2. "source_port" - the string name for the port of the source Part. The default port is "". There can be only one input port with a given name - input port names are distinct from one another. Likewise, there can be only one output port with a given name. Output ports are distinct from one another. Note that input ports and output ports are in different namespaces - hence, one input port might have the same name as one output port. 
3. "target_port" - the string name for the port of the receiver Part.
4. "source" - the string name of the template for the sender child Part.
5. "target" - the string name of the template for the receiver child Part.

A `down` connection does not specify a "source" Part. This implies that the the sender is the Container itself. In this case, there is a "source_port" which is an input port name of the Container.
An `up` connection does not specify a "receiver" Part. This implies that the the receiver is the Container itself. In this case, there is a "receiver_port" which is an output port name of the Container.
A `through` connection does not specify a "source" Part nor a "receiver" Part.



## sub-tasks to be described:
- register Container templates given a JSON array of Containers. At present, we use the drawio diagram editor to make drawings of Containers, one Container per tab on the drawio editor.
- register Leaf templates and their associated code.
- load a top-level Container, given an already-registered Container name and recursively instantiate all of its children and connection tables
- load a top-level Leaf, given an already-register Leaf name
- run - inject a mevent at the top level of a loaded Container
- concisely, give the specification so that it may be reused with other projects and other LLMs

# Appendix - Sample JSON for "crs.drawio" That Includes Several Containers
```
[
  {
    "name": "v0",
    "children": [
      {
        "name": "B",
        "id": 5
      },
      {
        "name": "C",
        "id": 9
      }
    ],
    "connections": [
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "B",
          "id": 5
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "B",
          "id": 5
        },
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "B",
          "id": 5
        }
      }
    ],
    "file": "crs.drawio"
  },
  {
    "name": "v1",
    "children": [
      {
        "name": "B",
        "id": 5
      },
      {
        "name": "C",
        "id": 9
      },
      {
        "name": "D",
        "id": 13
      }
    ],
    "connections": [
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "B",
          "id": 5
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "D",
          "id": 13
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "B",
          "id": 5
        },
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "D",
          "id": 13
        },
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "B",
          "id": 5
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "D",
          "id": 13
        }
      },
      {
        "dir": 3,
        "source_port": "✗",
        "target_port": "✗"
      },
      {
        "dir": 3,
        "source_port": "✗",
        "target_port": "✗"
      }
    ],
    "file": "crs.drawio"
  },
  {
    "name": "v2",
    "children": [
      {
        "name": "B",
        "id": 5
      },
      {
        "name": "C",
        "id": 9
      },
      {
        "name": "D",
        "id": 13
      }
    ],
    "connections": [
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "B",
          "id": 5
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "D",
          "id": 13
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "B",
          "id": 5
        },
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "D",
          "id": 13
        },
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "B",
          "id": 5
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "D",
          "id": 13
        }
      }
    ],
    "file": "crs.drawio"
  },
  {
    "name": "[dep] v2",
    "children": [
      {
        "name": "C",
        "id": 5
      },
      {
        "name": "D",
        "id": 9
      },
      {
        "name": "B",
        "id": 13
      }
    ],
    "connections": [
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "D",
          "id": 9
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "C",
          "id": 5
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "B",
          "id": 13
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "D",
          "id": 9
        },
        "target": {
          "name": "C",
          "id": 5
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "B",
          "id": 13
        },
        "target": {
          "name": "C",
          "id": 5
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "C",
          "id": 5
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "C",
          "id": 5
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "B",
          "id": 13
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "D",
          "id": 9
        }
      }
    ],
    "file": "crs.drawio"
  },
  {
    "name": "L0",
    "children": [
      {
        "name": "B",
        "id": 5
      },
      {
        "name": "L1",
        "id": 9
      }
    ],
    "connections": [
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "B",
          "id": 5
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "L1",
          "id": 9
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "B",
          "id": 5
        },
        "target": {
          "name": "L1",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "L1",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "L1",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "B",
          "id": 5
        }
      }
    ],
    "file": "crs.drawio"
  },
  {
    "name": "L1",
    "children": [
      {
        "name": "D",
        "id": 8
      },
      {
        "name": "L2",
        "id": 12
      },
      {
        "name": "D",
        "id": 22
      },
      {
        "name": "L2",
        "id": 26
      }
    ],
    "connections": [
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "D",
          "id": 8
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "L2",
          "id": 12
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "D",
          "id": 22
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "L2",
          "id": 26
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "D",
          "id": 8
        },
        "target": {
          "name": "L2",
          "id": 12
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "D",
          "id": 22
        },
        "target": {
          "name": "L2",
          "id": 26
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "L2",
          "id": 12
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "L2",
          "id": 12
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "D",
          "id": 8
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "L2",
          "id": 26
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "L2",
          "id": 26
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "D",
          "id": 22
        }
      },
      {
        "dir": 3,
        "source_port": "✗",
        "target_port": "✗"
      }
    ],
    "file": "crs.drawio"
  },
  {
    "name": "L2",
    "children": [
      {
        "name": "C",
        "id": 11
      },
      {
        "name": "C",
        "id": 22
      },
      {
        "name": "C",
        "id": 26
      }
    ],
    "connections": [
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "C",
          "id": 11
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "C",
          "id": 22
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "C",
          "id": 26
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "C",
          "id": 11
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "C",
          "id": 11
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "C",
          "id": 22
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "C",
          "id": 22
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "C",
          "id": 26
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "C",
          "id": 26
        }
      },
      {
        "dir": 3,
        "source_port": "✗",
        "target_port": "✗"
      },
      {
        "dir": 3,
        "source_port": "✗",
        "target_port": "✗"
      }
    ],
    "file": "crs.drawio"
  },
  {
    "name": "Video Base V1",
    "children": [
      {
        "name": "B",
        "id": 5
      },
      {
        "name": "C",
        "id": 9
      },
      {
        "name": "D",
        "id": 13
      }
    ],
    "connections": [
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "B",
          "id": 5
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 0,
        "source_port": "",
        "target_port": "",
        "target": {
          "name": "D",
          "id": 13
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "B",
          "id": 5
        },
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 1,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "D",
          "id": 13
        },
        "target": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "",
        "target_port": "",
        "source": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "C",
          "id": 9
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "B",
          "id": 5
        }
      },
      {
        "dir": 2,
        "source_port": "✗",
        "target_port": "✗",
        "source": {
          "name": "D",
          "id": 13
        }
      }
    ],
    "file": "crs.drawio"
  },
  {
    "name": "q",
    "children": [],
    "connections": [],
    "file": "crs.drawio"
  }
]
```
