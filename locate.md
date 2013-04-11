<small>	
	<a href="locate_docs.md">Documentation</a> |
	<a href="limitations.md">Limitations</a> |
	<a href="interpreting.md">Interpreting results</a>
</small>

<div id="location">	
	<div class="alert" id="statusMessage">Loading...</div>	
	<div class="row">
		<div class="span1">Location:</div>
		<div class="span2"><code><span id="LatitudeDisplay"></span> <span id="LongitudeDisplay"></span></code></div>	
	</div>
	<div class="row">
		<div class="span1">UTM: </div>
		<div class="span2"><code><span id="zone"></span> <span id="xPos"></span> <span id="yPos"></span>  </code></div>	
	</div>
	<div class="row">
		<div class="span1">Accuracy:</div>
		<div class="span2"><code><span id="AccuracyDisplay"></span> meters          </code></div>	
	</div>
	<span id="AgentDisplay" style="display:none" ></span>
</div>
<br/>
<iframe name="hidden_iframe" id="hidden_iframe" style="display:none;"></iframe>
<fieldset>
<legend>Submit results for testing:</legend>
<p class="muted">You can help us test this service by submitting your name/handle and a comment indicating your actual location</p>
	<form class="form-inline"
		method="post" action="https://script.google.com/macros/s/AKfycbz11NN8ls7rMUWIYS3qCkNskHQL4IoEtDw7fH-5BjHl6tBIUQo/exec" name="theForm" id="theForm" target="hidden_iframe" id="GoogleForm">  
		<input type="hidden" name="ID" id="ID" />  
		<input type="hidden" name="Latitude" id="Latitude" />  
		<input type="hidden" name="Longitude" id="Longitude"/>  
		<input type="hidden" name="UTM" id="UTM"/>  
		<input type="hidden" name="Accuracy" id="Accuracy"/>  
		<input type="hidden" name="Agent" id="Agent"/>  
		<input type="hidden" name="IPAddress" id="IPAddress"/>  
		<input type="hidden" name="Version" id="Version"/>  
		<input type="hidden" name="SubmitType" id="SubmitType"/>  
		<input type="text" name="Email" id="Email" placeholder="Name / handle" />  
		<input type="text" name="Comment" id="Comment" placeholder="Comment / actual location" />  
		<br/>
		<br/>
			<input type="button" value="Submit" class="submit btn" onclick="checkBeforeSubmit();" />	
			<input type="button" value="Reload" class="other btn" onclick="reload();" />	
			<span id="success"></span>  
	</form>
</fieldset>

<p> 
	<a id="mapsLink" title="Show on map" href="#">Map</a> | 	
	<a id="emailLink" title="Email this" href="#">Email</a> |	
	<a id="facebookLink" title="Share on Facebook" href="#">Facebook</a> |	
	<a id="tweetLink" title="Share on Twitter" href="#">Tweet</a> |	
	<a id="plusLink" title="Share on Google+" href="#">Google+</a>
</p>

<script type="text/javascript" src="js/geog.js"></script>
<script type="text/javascript" src="js/location.js"></script>
<script src="js/vendor/jquery.cookie.js"></script>
<script>
	$(document).ready(function () {		
		$('#emailLink').click(function(){ EmailPosition(); return false; });		
		$('#mapsLink').click(function(){ ShowLocation(); return false; });		
		$('#facebookLink').click(function(){ FacebookLocation(); return false; });		
		$('#tweetLink').click(function(){ TweetLocation(); return false; });	
		$('#plusLink').click(function(){ PlusLocation(); return false; });		
		$("#AgentDisplay").text(navigator.userAgent);		
		$("#Agent").val(navigator.userAgent);
		
		$("#VersionDisplay").text(LOCATION_VERSION);		
		$("#Version").val(LOCATION_VERSION);
		if (window['google'] && google.loader.ClientLocation) {			
			$("#LatitudeDisplay").text(google.loader.ClientLocation.latitude);			
			$("#LongitudeDisplay").text(google.loader.ClientLocation.longitude);		}
		setCookie();
		reload();	
	});	
	
	function setCookie()
	{
		try
		{
			if(!$.cookie('userID'))
			{
				var g=guid();
				$.cookie('userID', g, { expires: 365, path: '/' });	
			}
			$('#ID').val($.cookie('userID'));
		}
		catch(error)
		{
			console.log(error);
		}
	}
	
	function reload()	{
		//$("#statusMessage").toggle();
		$("#statusMessage").removeClass("alert-error");
		$("#statusMessage").removeClass("alert-success");
		$("#success").text("");		
		$("#statusMessage").text("Loading...");
		//prepareGeolocation();		
		doGeolocation();	
	}	
		
	function doGeolocation() {		
		if (navigator.geolocation) {			
			navigator.geolocation.getCurrentPosition(
				positionSuccess, 
				positionError,
				{timeout:10000, enableHighAccuracy:true});		
		} else {			
			positionError(-1);		
		}	
	}	

	function positionError(err) {		
		var msg;		
		switch(err.code) {
			case err.UNKNOWN_ERROR: msg = "Unable to find your location"; break;			
			case err.PERMISSION_DENINED: msg = "Permission denied in finding your location"; break;
			case err.POSITION_UNAVAILABLE: msg = "Your location is currently unknown"; break;			
			case err.TIMEOUT: msg = "Attempt to find location took too long"; break;			
			case err.BREAK: msg = "Attempt to find location took too long 2"; break;			
			default: msg = "Location detection not supported in browser";		
			}		
		$("#statusMessage").text(msg);		
		$("#statusMessage").addClass("alert-error");
		$("#Message").val("The position could not be determined.\nThe message was" + "\"" + msg + "\" (\" " + err.message + "\")");	
	}	
	
	function positionSuccess(position) {
		var coords = position.coords || position.coordinate || position;		
		$("#LatitudeDisplay").text(coords.latitude.toFixed(5));		
		$("#LongitudeDisplay").text(coords.longitude.toFixed(5));		
		$("#AccuracyDisplay").text(coords.accuracy);		
		$("#Latitude").val(coords.latitude);		
		$("#Longitude").val(coords.longitude);		
		$("#Accuracy").val(coords.accuracy);				
		convertToUTM(coords.latitude,coords.longitude);		
		$("#statusMessage").text("Location found on " + new Date());	
		$("#statusMessage").addClass("alert-success");
		try{
			doSubmit();
		}
		catch(err)
		{
		}
	}	

	function ShowLocation() {		
		window.open(GetShowLocationUrl() ,'_blank')	
	}	
	
	function FacebookLocation()	{		
		var href = "https://www.facebook.com/sharer/sharer.php?u=" + encodeURIComponent(GetShowLocationUrl());		
		window.open(href ,'_blank')	}	

	function TweetLocation()	{		
		var href = "https://twitter.com/intent/tweet?url=" + encodeURIComponent(GetShowLocationUrl());		
		window.open(href ,'_blank')	}	

	function PlusLocation()	{			
		var href = "https://plus.google.com/share?url=" + encodeURIComponent(GetShowLocationUrl());		
		window.open(href ,'_blank')	}	
	
	function doSubmit(){
		$("#SubmitType").val("Auto");
		document.getElementById('theForm').submit();		
		document.getElementById('success').innerHTML='Submitted'; }
	
	function checkBeforeSubmit(){		
		$("#SubmitType").val("Manual");
		//doSubmit();	
		document.getElementById('theForm').submit();		
		document.getElementById('success').innerHTML='Submitted'; 		
		alert("Your location has been submitted");	}			
		
	function convertToUTM(lat, lon)	{		
		var xy = new Array(2);		
		// Compute the UTM zone.		
		var zone = Math.floor ((lon + 180.0) / 6) + 1;		
		zone = LatLonToUTMXY (DegToRad (lat), DegToRad (lon), zone, xy);		
		$("#zone").text(zone);		
		$("#xPos").text(xy[0].toFixed(0));		
		$("#yPos").text(xy[1].toFixed(0));				
		$("#UTM").val(zone 			
			+ " " 			
			+ xy[0].toFixed(0) 			
			+ " " 			
			+ xy[1].toFixed(0));	
	}
</script>