using Chuon;
using JimmikerNetwork;
using JimmikerNetwork.Server;
using System;
using System.Collections;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;

namespace WarFightGameServer
{
    public class Appllication : AppllicationBase
    {
        public event Action<MessageType, string> GetMessage;

        public ClientList ClientList { get; private set; }
        public RoomList RoomList { get; private set; }

        public Dictionary<string, object> DataBase { get; private set; }

        public IPEndPoint PublicEndPoint { get; private set; }

        public float deltaTime { get; private set; }
        Stopwatch updatetime;
        public Appllication(string host, int port, ProtocolType protocol) : base(IPAddress.IPv6Any, port, protocol)
        {
            if (string.IsNullOrEmpty(host))
            {
                using (WebClient webClient = new WebClient())
                {
                    host = webClient.DownloadString("http://ifconfig.me");
                }
            }
            PublicEndPoint = new IPEndPoint(IPAddress.Parse(host), port);
        }

        protected override void Setup()
        {
            updatetime = new Stopwatch();
            ClientList = new ClientList((id) => !string.IsNullOrEmpty(id));
            RoomList = new RoomList();
            ChuonBinary chuonBinary = new ChuonBinary(File.ReadAllBytes("database.dat"));
            DataBase = (Dictionary<string, object>)chuonBinary.ToObject();
            
            RunUpdateThread();
        }

        protected override PeerBase AddPeerBase(object _peer, INetServer server)
        {
            return new Peer(_peer, server, this);
        }

        protected override void TearDown()
        {
            
        }

        protected override void DebugReturn(MessageType messageType, string msg)
        {
            GetMessage?.Invoke(messageType, msg);
        }

        public override void Update()
        {
            updatetime.Stop();
            deltaTime = updatetime.ElapsedMilliseconds / 1000f;
            updatetime.Restart();

            base.Update();
            foreach (Room room in RoomList)
            {
                room.Update();
            }
            if(Program.client != null)
                Program.client.ClientLinker.Update();
        }
    }
}# Stopwatch-web-application.java
