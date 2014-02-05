using UnityEngine;
using System;
using System.Collections;
using System.Net;
using System.Net.Sockets;
using System.Threading;
using System.Collections.Generic;

public class client_class{ 
	public Socket connection = null;
	public const int buff_size = 256;
	public byte[] buff = new byte[buff_size];
}

public class Client : MonoBehaviour {

	static public double data;

	static List<int> decode_data_int(int decode_data_int){
		//ints
		List<int> digits = new List<int>();
		int decode_data_int_copy = decode_data_int;
		bool found = false;
		while(!found){
			int places = 0;
			while(decode_data_int_copy > 10){
				decode_data_int_copy /= 10;
				places++;
			}
			if(places == 0){
				found = true;
			}
			digits.Add(decode_data_int_copy);
			decode_data_int -= (int)(decode_data_int_copy * Math.Pow(10, places));
			decode_data_int_copy = decode_data_int;
		}
		return digits;
	}

	static List<int> decode_data_double(byte[] data){
		//doubles
		int places = 0;
		double decode_data_double = Convert.ToDouble(data);
		double decode_data_copy = decode_data_double;
		while(decode_data_copy - (int)decode_data_copy > 0.1){
			decode_data_copy *= 10;
			places++;
		}
		decode_data_copy = decode_data_double;
		decode_data_copy -= (int)decode_data_copy;
		decode_data_copy *= Math.Pow(10, places);
		return decode_data_int((int)decode_data_copy);
	}

	IEnumerator pause() {
		yield return new WaitForEndOfFrame();
	}

	private static ManualResetEvent connectDone = new ManualResetEvent(false); //event state
	private static ManualResetEvent sendDone = new ManualResetEvent(false); //event state

	void client(){
		//IPAddress extern_ip = ""; //gets the local IP
		//IPEndPoint externEndPoint = new IPEndPoint(extern_ip, 0); //gets the local IP
		Socket connection = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
		//client.BeginConnect(externEndPoint, new AsyncCallback(ConnectCallback), client);

		client_class client = new client_class();
		client.connection = connection;

		connection.BeginReceive(client.buff, 0, client_class.buff_size, 0, new AsyncCallback(ReceiveCallback), client);
		while(true){
			if(data != -1){
				byte[] byteData = BitConverter.GetBytes(data);
				connection.BeginSend(byteData, 0, byteData.Length, SocketFlags.None, new AsyncCallback(SendCallback), connection);
			}
			StartCoroutine(pause()); //to ease load of while loop. possibly too long of a pause
		}
	}

	private static void ConnectCallback(IAsyncResult ar){
		// Retrieve the socket from the state object.
		Socket connection = (Socket)ar.AsyncState;
		
		// Complete the connection.
		connection.EndConnect(ar);
		
		// Signal that the connection has been made.
		connectDone.Set();
	}

	private static void SendCallback(IAsyncResult ar){
		// Retrieve the socket from the state object.
		Socket connection = (Socket)ar.AsyncState;
		
		// Complete sending the data to the remote device.
		connection.EndSend(ar);

		data = -1;
		
		// Signal that all bytes have been sent.
		sendDone.Set();
	}
	
	private static void ReceiveCallback(IAsyncResult ar){
		// Retrieve the state object and the client socket 
		// from the asynchronous state object.
		client_class client = (client_class)ar.AsyncState;
		Socket connection = client.connection;
		// Read data from the remote device.
		int bytesRead = connection.EndReceive(ar);
		if (bytesRead > 0) {
			//todo: process decoded data using dictionary
			List<int> data_deci = decode_data_double(client.buff);
			List<int> data_int = decode_data_int((int)Convert.ToDouble(client.buff));
			//Get the rest of the data.
			connection.BeginReceive(client.buff, 0, client_class.buff_size, 0, new AsyncCallback(ReceiveCallback), client);
		}
	}

	// Use this for initialization
	void Start(){
		client();
	}
	
	// Update is called once per frame
	void Update(){
	}
}
