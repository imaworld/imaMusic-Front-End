<script>
	export default{
		created(){
			localStorage.removeItem('Token')
			localStorage.setItem("SetTimer", null)
        	window.location.href = '/musicify/login'
		}
	};
</script>