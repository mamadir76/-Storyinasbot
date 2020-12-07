<?php 
ini_set('memory_limit', '1024M'); 
define('API_KEY','426917934:AAEA4WMCIcH4u-LgaOrNvtuxX4F9PdtruqE'); 
$telegram = json_decode(file_get_contents('php://input'),true);
$user_id = $telegram['message']['chat']['id'];
$url = $telegram['message']['text'];


	if($url == "/start"){
		bot(
		'sendMessage', [
			'chat_id'=> $user_id,
			'text'=> 'سلام خوش آمدید . لطفا لینک فایل مورد نظر را ارسال کنید .',
		]);		
	}else{
			
		if(filter_var($url, FILTER_VALIDATE_URL)){
			bot('sendMessage', ['chat_id'=> $user_id,'text'=> 'در حال آپلود فایل ...']);
			if(remote_file_size($url) < 50){
				$fileName = upload($url);	
				send_file( $user_id , $fileName);
				bot('sendMessage', ['chat_id'=> $user_id,'text'=> 'https://tooba.co/files/'.$fileName ]);
			}	
				
			
		}
				
	}



	function bot($method,$datas=[]){
		 $url = "https://api.telegram.org/bot".API_KEY."/".$method; $ch = curl_init();
		  curl_setopt($ch,CURLOPT_URL,$url); 
		  curl_setopt($ch,CURLOPT_RETURNTRANSFER,true); 
		  curl_setopt($ch,CURLOPT_POSTFIELDS,$datas); 
		  $res = curl_exec($ch); 
		  if(curl_error($ch)){
			var_dump(curl_error($ch)); 
		  }else{ 
			return json_decode($res); 
		  } 
	}
	
	
	function remote_file_size($url){
		 $ch = curl_init($url);

		 curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
		 curl_setopt($ch, CURLOPT_HEADER, TRUE);
		 curl_setopt($ch, CURLOPT_NOBODY, TRUE);

		 $data = curl_exec($ch);
		 $size = curl_getinfo($ch, CURLINFO_CONTENT_LENGTH_DOWNLOAD);

		 curl_close($ch);
		 return round(($size/1024)/1024);
	}
	
	


	function upload($url){
		 $filename= preg_replace('/\\?.*/', '', basename($url));
		 $to = "files/".$filename;
		 $data=file_get_contents($url);
		 if($data===false) 
			return false;
		 else{	
			file_put_contents($to,$data);
			return $filename;
		}		
	}
	

	function send_file( $user_id , $fileName){
			
		$url= "https://api.telegram.org/bot".API_KEY."/sendDocument?chat_id=$user_id";
		$post = array(
		 "document"  => new CURLFile(realpath('files/'.$fileName))
		);
		$ch = curl_init();
		curl_setopt($ch, CURLOPT_URL, $url);
		curl_setopt($ch, CURLOPT_POSTFIELDS, $post);
		curl_exec($ch);
	}	 
