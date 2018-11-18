Real-Time Analysis TODO
=======================

 - More testing!  This is not straightforward, since testing the real-time server in a variety of
   use cases is nontrivial.  We probably want to do some groundwork first, in order to make tests 
   easier to write.  For example, it would be really helpful to define python wrappers for the L1 
   server/client, and helper functions for writing unit tests in python.

 - Shift central frequencies, as suggested by Ziggy and Kiyo's analysis of the frequency response
   of our upchannelization scheme.  (This is a trivial change, but we may want to wait until our
   October 27 sensitivity issues are resolved first, as part of a change-one-thing-at-a-time philosophy.)

 - Decrease latency of L1 server.  (Masoud and Kendrick are currently working on this.)

 - Real-time pulse injection.  (Dustin has PRs in for this.)

 - Generate timestreams for pulsar search (Alex Roman is currently working on this.)

 - Add RFI mask fraction statistics to Prometheus/Grafana.  (Done; https://grafana.chimenet.ca/d/7zzUhPfiz/l1-rfi-masking?refresh=5m&orgId=1)

 - Implement an RFI ultra-coarse-grained viewer, now that we have the "plumbing" in place.

 - Collect statistics for pipeline latency at a few places in the pipeline.  (Dustin has PRs in for this.)

 - Collect statistics for "idle fraction" of different threads in the pipeline.  (Assembler and network threads have this already.)
   A nice impementation could be
   to define a std::condition_variable wrapper class which assumes a single waiter, automatically keeps track
   of the waiter's idle fraction, and allows the idle fraction to be queried by an arbitrary thread.  (But,
   if we use this approach, I think we need to switch some pthreads to std::thread.  See "cleanup" item below!)

 - Logging and capturing error messages.  We have a multithreaded logging system (chlog) but we're not
   currently using it consistently (sometimes we call chlog() and sometimes we just use cout).  What should
   we be doing?  (We now run the L1 server as a "service", does this change anything?)

   In practice, the most important case is where the L1 server crashes, and we want to know why.  Usually
   a crash is accompanied by an exception, so we just want to make sure that the exception text gets captured
   and is easily accessible through the high-level webapp.  (It's possible that we're already doing this, and
   I'm just out of the loop :))

 - If the L1 server runs out of memory, then it currently crashes!  This is currently its main failure mode
   (in the typical failure scenario, there is a long-duration RFI storm which triggers continuous write_requests,
   which overload the NFS server and generate a backlog of write_requests on the L1 nodes).

   Instead, the L1 server should cancel write requests in order to reclaim memory, ordered from lowest priority
   to highest.

 - Cleanup/reorganization: factor python code in the ch-frb-l1 repo to its own git repository.  This was requested
   by the McGill team (and seems like a good idea in general, since python users shouldn't need to compiling the C++ server
   and all of its prerequisites).  We should coordinate with everyone on details such as, do we move the webapp to the
   new repo (or would it make more sense to make another new repo for the webapp?)  We should also check to make
   sure we're not introducing conflicts on feature branches (e.g. I think Alex Roman is editing the python RPC client
   on his slow pulsar feature branches).

   Related: currently many python source files are duplicated in the source tree (e.g. ``ch_frb_l1/cnc_ssh.py`` and
   ``ch_frb_l1/webapp/cnc_ssh.py`` are copies of each other).

 - Cleanup: add documentation for all RPC's.  (There is a placeholder section in the manual here: https://kmsmith137.github.io/ch_frb_l1_docs/server/rpc_reference.html)  At the moment, the only documentation is in https://github.com/kmsmith137/ch_frb_l1/blob/master/rpc.hpp)

 - Cleanup: command-line parsing in ch-frb-l1 needs improvement!  What happened here is that we started out with
   "homegrown" command-line parsing, but eventually outgrew it.  We then switched to cxxopts (https://github.com/jarro2783/cxxopts)
   and learned at the last minute that this didn't work on the production L1 nodes, since gcc 4.8 doesn't implement the C++11 regexp
   library.  Currently the code is a homegrown-cxxopts hybrid which isn't very maintainable.

   I think the path of least resistance is to switch again, to CLI11 (https://github.com/CLIUtils/CLI11) which
   explicitly mentions supporting gcc 4.8 as a design goal.  I think this shouldn't be too time-consuming but I
   understand why no one is excited about doing it!  :)

 - Cleanup: switch from pthreads to std::thread.  (Dustin has a PR in for this.)

   The C++11 std::thread API is a lot better than the vintage-1980 pthread API.  (The only reason we still
   use pthreads is that I started out using C++03 instead of C++11, and this never got fully cleaned up!)

 - Cleanup: general reorganization of ch_frb_l1, and factoring into more functions.

   This is more of a proposal than a well-defined todo item!  The ch_frb_l1 code has grown by incremental hacking,
   and could benefit from a cleanup pass.  (Random example: L1RpcServer::_handle_request() is currently 500
   lines, and is begging to be factored into one function for each RPC.)

   I think this is important for long-term maintainability, but we should postpone until all major features
   are implemented (a milestone which is not too far away!)
