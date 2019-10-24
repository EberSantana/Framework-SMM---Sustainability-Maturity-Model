# Framework-SMM---Sustainability-Maturity-Model
Fonte das classes mais importantes do MVP
MVC
Controller
PesquisaController.php
<?php

	class PesquisaController
	{
		public function carregar()
		{
			try {
				$colecPostagens = Pesquisa::carregar();

				$loader = new \Twig\Loader\FilesystemLoader('app/View');
				$twig = new \Twig\Environment($loader);
				$template = $twig->load('home.html');

				$parametros = array();
				$parametros['pesquisa'] = $colecPostagens;
				$enc = array_map("utf8_encode",$parametros);
				#var_dump($parametros['pesquisa'],$enc);

				$conteudo = $template->render($parametros);
				#echo $conteudo;
				#echo($parametros);
				print $colecPostagens;

				
				
			} catch (Exception $e) {
				echo $e->getMessage();
			}
			
		}
		public function inserir()
		{
			try {
				Pesquisa::insert($_POST);

				echo '<script>alert("Pesquisa inserida com sucesso!");</script>';
				
			} catch(Exception $e) {
				echo '<script>alert("'.$e->getMessage().'");</script>';
				
			}
			
		}
		
		
	}

PainelController.php
<?php

	class PainelController
	{
		public function consulta()
		{
			try {
				$colecPostagens = Painel::painel_geral();

				$loader = new \Twig\Loader\FilesystemLoader('app/View');
				$twig = new \Twig\Environment($loader);
				$template = $twig->load('home.html');

				$parametros = array();
				$parametros['pesquisa'] = $colecPostagens;
				$enc = array_map("utf8_encode",$parametros);
				#var_dump($parametros['pesquisa'],$enc);

				$conteudo = $template->render($parametros);
				#echo $conteudo;
				#echo($parametros);
				print $colecPostagens;

				
				
			} catch (Exception $e) {
				echo $e->getMessage();
			}
			
		}
		
		public function consulta_resumo()
		{
			try {
				$colecPostagens = Painel::painel_resumo();

				$loader = new \Twig\Loader\FilesystemLoader('app/View');
				$twig = new \Twig\Environment($loader);
				$template = $twig->load('home.html');

				$parametros = array();
				$parametros['pesquisa'] = $colecPostagens;
				$enc = array_map("utf8_encode",$parametros);
				#var_dump($parametros['pesquisa'],$enc);

				$conteudo = $template->render($parametros);
				#echo $conteudo;
				#echo($parametros);
				print $colecPostagens;

				
				
			} catch (Exception $e) {
				echo $e->getMessage();
			}
			
		}
	}
Model
Pesquisa.php
<?php
	header("Content-Type: text/html; charset=ISO-8859-1", true); 
	class Pesquisa
	{

		public static function carregar()
		{
			$con = Connection::getConn();

			$sql = "select d.descr Dominio,d.id Id_Dominio,s.descr Subdominio, s.id Id_Subdominio,i.descr Indicadores, i.id Id_indicadores, d.pd pd, s.psd psd
					from dominio d
					INNER JOIN subdominio s on s.id_dominio = d.id 
					INNER join indicador i on i.id_subdominio = s.id";
			$sql = $con->prepare($sql);

			$sql->execute();

			$resultado = array();
			
			while ($row = $sql->fetchObject('pesquisa'))
			{
				$resultado[] = $row;
			}
			#utf8_encode($resultado);
			#var_dump($resultado);
		

			if (!$resultado) {
				throw new Exception("Não foi encontrado nenhum registro no banco");		
			}
			
			

			$dados=json_encode($resultado,JSON_UNESCAPED_UNICODE|JSON_INVALID_UTF8_IGNORE);

			return $dados;
		}



		public static function selecionaPorId($idPost)
		{
			$con = Connection::getConn();

			$sql = "SELECT * FROM pesquisa WHERE id = :id";
			$sql = $con->prepare($sql);
			$sql->bindValue(':id', $idPost, PDO::PARAM_INT);
			$sql->execute();

			$resultado = $sql->fetchObject('pesquisa');

			if (!$resultado) {
				throw new Exception("Não foi encontrado nenhum registro no banco");	
			} else {
				$resultado->comentarios = Comentario::selecionarComentarios($resultado->id);
			}

			return $resultado;
		}

		public static function insert($params)
		{
			// if (empty($dadosPost['titulo']) OR empty($dadosPost['conteudo'])) {
				// throw new Exception("Preencha todos os campos");

				// return false;
			// }

			$con = Connection::getConn();

			$sql = $con->prepare('INSERT INTO pesquisa (id_indicador, valor,ano,id_cidade) VALUES (:id, :valor,:ano,:cidade)');
			$sql->bindValue(':id', $params['id']);
			$sql->bindValue(':valor', $params['valor']);
			$sql->bindValue(':ano', $params['ano']);
			$sql->bindValue(':cidade', $params['cidade']);
			$res = $sql->execute();

			if ($res == 0) {
				throw new Exception("Falha ao inserir pesquisa");

				return false;
			}

			return true;
		}

		public static function update($params)
		{
			$con = Connection::getConn();

			$sql = "UPDATE pesquisa SET id_cidade = :cidade, ano = :ano, valor = :valor WHERE id_indicador = :id";
			$sql = $con->prepare($sql);
			$sql->bindValue(':cidade', $params['cidade']);
			$sql->bindValue(':ano', $params['ano']);
			$sql->bindValue(':valor', $params['valor']);
			$sql->bindValue(':id', $params['id']);
			$resultado = $sql->execute();

			if ($resultado == 0) {
				throw new Exception("Falha ao alterar pesquisa");

				return false;
			}

			return true;
		}

		public static function delete($id)
		{
			$con = Connection::getConn();

			$sql = "DELETE FROM pesquisa WHERE id = :id";
			$sql = $con->prepare($sql);
			$sql->bindValue(':id', $id);
			$resultado = $sql->execute();

			if ($resultado == 0) {
				throw new Exception("Falha ao deletar pesquisa");

				return false;
			}

			return true;
		}

	}




Painel.php
<?php
	header("Content-Type: text/html; charset=ISO-8859-1", true); 
	class Painel
	{


		public static function painel_geral()
		{
			$con = Connection::getConn();
			
			$sql = "SELECT *  FROM painel";
			$sql = $con->prepare($sql);
			#$resultado = mysqli_query($con,$sql);
			#$sql -> mysql_query("SET CHARACTER SET utf8");
			$sql->execute();

			$resultado = array();

			while ($row = $sql->fetchObject('pesquisa'))
			{
				$resultado[] = $row;
			}

		
			if (!$resultado) {
				throw new Exception("Não foi encontrado nenhum registro no banco");		
			}

			$dados=json_encode($resultado,JSON_UNESCAPED_UNICODE|JSON_INVALID_UTF8_IGNORE);

			return $dados;
		}
		
		public static function painel_resumo()
		{
			$con = Connection::getConn();
			
			$sql = "SELECT *  FROM resumo_painel";
			$sql = $con->prepare($sql);

			$sql->execute();

			$resultado = array();

			while ($row = $sql->fetchObject('pesquisa'))
			{
				$resultado[] = $row;
			}

		
			if (!$resultado) {
				throw new Exception("Não foi encontrado nenhum registro no banco");		
			}

			$dados=json_encode($resultado,JSON_UNESCAPED_UNICODE|JSON_INVALID_UTF8_IGNORE);

			return $dados;
		}

		



	}

MVP
JS
Pesquisa.js
$(function () {
	url = "http://192.168.0.42";
    $(".detModal").hide();
    montatela();
   
});

$(".detFechar").live("click", function () {
    $(".detModal, .fundoModal").fadeOut();
})


function number_format(number, decimals, dec_point, thousands_sep) {
    var n = number, c = isNaN(decimals = Math.abs(decimals)) ? 2 : decimals;
    var d = dec_point == undefined ? "," : dec_point;
    var t = thousands_sep == undefined ? "." : thousands_sep, s = n < 0 ? "-" : "";
    var i = parseInt(n = Math.abs(+n || 0).toFixed(2)) + "", j = (j = i.length) > 3 ? j % 3 : 0;
    return s + (j ? i.substr(0, j) + t : "") + i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + t) + (c ? d + Math.abs(n - i).toFixed(2).slice(2) : "");
}

$("#salvar").live("click", function () {
    var ano = document.getElementById("cboAno").value;
	var cidade = document.getElementById("cboCidades").value;
	var urlp = url+"/mvc/?pagina=pesquisa&metodo=inserir";
	
	if (ano && cidade){
		var trList = document.querySelectorAll("#extrato-usuario tbody tr");
		
		for(var i = 0 ; i < trList.length ; i++)
		{
			let check = 0;
			let ind = trList[i]

            let tdId = ind.querySelector('#id_indicador').getAttribute("class")
			
			let id_indicador = tdId.substring(tdId.lastIndexOf(' ')+1);
			let rdvalor = 'valor_ind'+ id_indicador;
			//alert(rdvalor);
			var radios = document.getElementsByName(rdvalor);
			//alert(radios.length);
			for (var x = 0; x < radios.length; x++) {
				if (radios[x].checked) {
					check = 1;
					$.ajax({
						url: urlp,
						type: "POST",
						data: { id: id_indicador, valor: radios[x].value, ano: ano, cidade: cidade },
						dataType: "json",
						success: function (result) {
							alert(result);
						},
						error: function (xhr, ajaxOptions, thrownError) {
						alert(xhr.status);
						alert(thrownError);
						}
					});
				}
			}
			if(check == 0){
				alert('Marque todos os indicadores');
				break;
			}else{
				
				check = 0;
			}
			//alert(tdId.lastIndexOf(' '));
			//alert(tdId.substring(tdId.lastIndexOf(' ')));
		}
		
	}
	else{
		 alert("Selecione cidade e ano");
	}
	

})



function montatela() {


	var urlp = url+"/mvc/?pagina=pesquisa&metodo=carregar";
	                          
	$.ajax({
		url:        urlp,
		dataType:   "json", 
		success: function (data) {

			
			$("#gcd-pesquisa").append(
				'<table id="extrato-usuario" class="tablesorter">' +
					'<thead>' +
							'<tr>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">ICG</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">DOMINIO</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">ICD</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">SUBDOMINIO</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">IMSD</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">INDICADORES DA ISO</th>' +
								'<th class="gcd-extrato-centro icd">NA</th>' +
								'<th class="gcd-extrato-centro icd">PA</th>' +
								'<th class="gcd-extrato-centro icd">TA</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">PSD</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">PD</th>' +
					  
							'</tr>' +
							'<tr>' +
								
								'<th class="gcd-extrato-centro icd">0</th>' +
								'<th class="gcd-extrato-centro icd">1</th>' +
								'<th class="gcd-extrato-centro icd">2</th>' +
								
					  
							'</tr>' +
							
						'</thead>'+
						'<tbody>' +
						'</tbody>' +
							'</table>'


					);

		 
			$.each(data, function (i, item) {
									   
				
				if ( item.Id_Dominio == "1")
				{
					if (item.Id_indicadores == "1"){
						
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
							'<td class="gcd-extrato-centro icd" rowspan="75">' + ' ' + '</td>' +
							'<td class="ind3" rowspan="7">' + item.Dominio + '</td>' +
							'<td class="ind3" rowspan="7">' + ' ' + '</td>' +
							'<td class="ind3" rowspan="4">' + item.Subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind3" rowspan="4">' + number_format(0, 1, ',', '.') + '%</td>' +
							'<td class="ind3 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind3" rowspan="4">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3" rowspan="7">' + number_format(item.pd, 0, ',', '.') + '</td>' +
							'</tr>')
					}else{
						if (item.Id_indicadores == "5"){
							$("#extrato-usuario tbody").append(
							'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
								'<td class="ind3" rowspan="3">' + item.Subdominio + '</td>' +
								'<td class="gcd-extrato-centro percent ind3" rowspan="3">' + number_format(0, 1, ',', '.') + '%</td>' +
								'<td class="ind3 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
								'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind3" rowspan="3">' + number_format(item.psd, 0, ',', '.') + '</td>' +

				
								'</tr>')
						}
						else{
							$("#extrato-usuario tbody").append(
							'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
						
								'<td class="ind3 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
								'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind3">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
						

				
								'</tr>')

						}
					}
					
					
				}
				if ( item.Id_Dominio == "2")
				{
					if (item.Id_indicadores == "8"){
						
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
							'<td class="ind4" rowspan="5">' + item.Dominio + '</td>' +
							'<td class="ind4" rowspan="5">' + ' ' + '</td>' +
							'<td class="ind4" rowspan="5">' + item.Subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind4" rowspan="5">' + number_format(0, 1, ',', '.') + '%</td>' +
							'<td class="ind4 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind4" rowspan="5">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind4" rowspan="5">' + number_format(item.pd, 0, ',', '.') + '</td>' +
							'</tr>')
					}else{
						$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
						
						
							'<td class="ind4 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
					
							
							'</tr>')
						
					}
					
					
				}
				if ( item.Id_Dominio == "3")
				{
					if (item.Id_indicadores == "13"){
						
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
							'<td class="ind5" rowspan="5">' + item.Dominio + '</td>' +
							'<td class="ind5" rowspan="5">' + ' ' + '</td>' +
							'<td class="ind5" rowspan="5">' + item.Subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind5" rowspan="5">' + number_format(0, 1, ',', '.') + '%</td>' +
							'<td class="ind5 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind5" rowspan="5">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind5" rowspan="5">' + number_format(item.pd, 0, ',', '.') + '</td>' +
							'</tr>')
					}else{
						$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
						
						
							'<td class="ind5 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
					
							
							'</tr>')
						
					}
					
					
				}
				if ( item.Id_Dominio == "4")
				{
					if (item.Id_indicadores == "18"){
						
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
							'<td class="ind6" rowspan="14">' + item.Dominio + '</td>' +
							'<td class="ind6" rowspan="14">' + ' ' + '</td>' +
							'<td class="ind6" rowspan="4">' + item.Subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind6" rowspan="4">' + number_format(0, 1, ',', '.') + '%</td>' +
							'<td class="ind6 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind6" rowspan="4">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6" rowspan="14">' + number_format(item.pd, 0, ',', '.') + '</td>' +
							'</tr>')
					}else{
						if (item.Id_indicadores == "22"){
							$("#extrato-usuario tbody").append(
							'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
								'<td class="ind6" rowspan="10">' + item.Subdominio + '</td>' +
								'<td class="gcd-extrato-centro percent ind6" rowspan="10">' + number_format(0, 1, ',', '.') + '%</td>' +
								'<td class="ind6 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
								'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind6" rowspan="10">' + number_format(item.psd, 0, ',', '.') + '</td>' +

				
								'</tr>')
						}
						else{
							$("#extrato-usuario tbody").append(
							'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
						
								'<td class="ind6 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
								'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind6">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
						

				
								'</tr>')

						}
					}
					
					
				}
				if ( item.Id_Dominio == "5")
				{
					if (item.Id_indicadores == "32"){
						
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
							'<td class="ind7" rowspan="25">' + item.Dominio + '</td>' +
							'<td class="ind7" rowspan="25">' + ' ' + '</td>' +
							'<td class="ind7" rowspan="7">' + item.Subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind7" rowspan="7">' + number_format(0, 1, ',', '.') + '%</td>' +
							'<td class="ind7 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind7" rowspan="7">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7" rowspan="25">' + number_format(item.pd, 0, ',', '.') + '</td>' +
							'</tr>')
					}else{
						if (item.Id_indicadores == "39"){
							$("#extrato-usuario tbody").append(
							'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
								'<td class="ind7" rowspan="4">' + item.Subdominio + '</td>' +
								'<td class="gcd-extrato-centro percent ind7" rowspan="4">' + number_format(0, 1, ',', '.') + '%</td>' +
								'<td class="ind7 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
								'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind7" rowspan="4">' + number_format(item.psd, 0, ',', '.') + '</td>' +

				
								'</tr>')
						}
						else{
							if (item.Id_indicadores == "43"){
							$("#extrato-usuario tbody").append(
							'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
								'<td class="ind7" rowspan="3">' + item.Subdominio + '</td>' +
								'<td class="gcd-extrato-centro percent ind7" rowspan="3">' + number_format(0, 1, ',', '.') + '%</td>' +
								'<td class="ind7 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
								'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind7" rowspan="3">' + number_format(item.psd, 0, ',', '.') + '</td>' +

				
								'</tr>')
							}
							else{
								if (item.Id_indicadores == "46"){
									$("#extrato-usuario tbody").append(
									'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
										'<td class="ind7" rowspan="2">' + item.Subdominio + '</td>' +
										'<td class="gcd-extrato-centro percent ind7" rowspan="2">' + number_format(0, 1, ',', '.') + '%</td>' +
										'<td class="ind7 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
										'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
										'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
										'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
										'<td class="gcd-extrato-centro percent ind7" rowspan="2">' + number_format(item.psd, 0, ',', '.') + '</td>' +

						
										'</tr>')
								}
								else{
									if (item.Id_indicadores == "48"){
										$("#extrato-usuario tbody").append(
										'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
											'<td class="ind7" rowspan="3">' + item.Subdominio + '</td>' +
											'<td class="gcd-extrato-centro percent ind7" rowspan="3">' + number_format(0, 1, ',', '.') + '%</td>' +
											'<td class="ind7 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
											'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
											'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
											'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
											'<td class="gcd-extrato-centro percent ind7" rowspan="3">' + number_format(item.psd, 0, ',', '.') + '</td>' +

							
											'</tr>')
									}else{
										if (item.Id_indicadores == "51"){
											$("#extrato-usuario tbody").append(
											'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
												'<td class="ind7" rowspan="4">' + item.Subdominio + '</td>' +
												'<td class="gcd-extrato-centro percent ind7" rowspan="4">' + number_format(0, 1, ',', '.') + '%</td>' +
												'<td class="ind7 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
												'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind7" rowspan="4">' + number_format(item.psd, 0, ',', '.') + '</td>' +

								
												'</tr>')
										}else{
											if (item.Id_indicadores == "55"){
											$("#extrato-usuario tbody").append(
											'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
												'<td class="ind7" rowspan="2">' + item.Subdominio + '</td>' +
												'<td class="gcd-extrato-centro percent ind7" rowspan="2">' + number_format(0, 1, ',', '.') + '%</td>' +
												'<td class="ind7 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
												'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind7" rowspan="2">' + number_format(item.psd, 0, ',', '.') + '</td>' +

								
												'</tr>')
											}else{
											
											
												$("#extrato-usuario tbody").append(
												'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
											
													'<td class="ind7 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
													'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
													'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
													'<td class="gcd-extrato-centro percent ind7">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
											

									
													'</tr>')
											}

										}
									}
					
					
								}
							}
				
						}
				
		
					}
				}
				if ( item.Id_Dominio == "6")
				{
					if (item.Id_indicadores == "57"){
						
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
							'<td class="ind8" rowspan="19">' + item.Dominio + '</td>' +
							'<td class="ind8" rowspan="19">' + ' ' + '</td>' +
							'<td class="ind8" rowspan="3">' + item.Subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind8" rowspan="3">' + number_format(0, 1, ',', '.') + '%</td>' +
							'<td class="ind8 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
							'<td class="gcd-extrato-centro percent ind8" rowspan="3">' + number_format(item.pd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8" rowspan="19">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'</tr>')
					}else{
						if (item.Id_indicadores == "60"){
							$("#extrato-usuario tbody").append(
							'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
								'<td class="ind8" rowspan="4">' + item.Subdominio + '</td>' +
								'<td class="gcd-extrato-centro percent ind8" rowspan="4">' + number_format(0, 1, ',', '.') + '%</td>' +
								'<td class="ind8 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
								'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind8" rowspan="4">' + number_format(item.psd, 0, ',', '.') + '</td>' +

				
								'</tr>')
						}
						else{
							if (item.Id_indicadores == "64"){
							$("#extrato-usuario tbody").append(
							'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
								'<td class="ind8" rowspan="3">' + item.Subdominio + '</td>' +
								'<td class="gcd-extrato-centro percent ind8" rowspan="3">' + number_format(0, 1, ',', '.') + '%</td>' +
								'<td class="ind8 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
								'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
								'<td class="gcd-extrato-centro percent ind8" rowspan="3">' + number_format(item.psd, 0, ',', '.') + '</td>' +

				
								'</tr>')
							}
							else{
								if (item.Id_indicadores == "67"){
									$("#extrato-usuario tbody").append(
									'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
										'<td class="ind8" rowspan="4">' + item.Subdominio + '</td>' +
										'<td class="gcd-extrato-centro percent ind8" rowspan="4">' + number_format(0, 1, ',', '.') + '%</td>' +
										'<td class="ind8 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
										'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
										'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
										'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
										'<td class="gcd-extrato-centro percent ind8" rowspan="4">' + number_format(item.psd, 0, ',', '.') + '</td>' +

						
										'</tr>')
								}
								else{
									if (item.Id_indicadores == "71"){
										$("#extrato-usuario tbody").append(
										'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
											'<td class="ind8" rowspan="2">' + item.Subdominio + '</td>' +
											'<td class="gcd-extrato-centro percent ind8" rowspan="2">' + number_format(0, 1, ',', '.') + '%</td>' +
											'<td class="ind8 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
											'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
											'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
											'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
											'<td class="gcd-extrato-centro percent ind8" rowspan="2">' + number_format(item.psd, 0, ',', '.') + '</td>' +

							
											'</tr>')
									}else{
										if (item.Id_indicadores == "73"){
											$("#extrato-usuario tbody").append(
											'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
												'<td class="ind8" rowspan="3">' + item.Subdominio + '</td>' +
												'<td class="gcd-extrato-centro percent ind8" rowspan="3">' + number_format(0, 1, ',', '.') + '%</td>' +
												'<td class="ind8 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
												'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
												'<td class="gcd-extrato-centro percent ind8" rowspan="3">' + number_format(item.psd, 0, ',', '.') + '</td>' +

								
												'</tr>')
										}else{

												$("#extrato-usuario tbody").append(
												'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.Dominio + '">' +
											
													'<td class="ind8 '+item.Id_indicadores+'" id="id_indicador">' + item.Indicadores + '</td>' +
													'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="0"/>' + '</td>' +
													'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="1"/>' + '</td>' +
													'<td class="gcd-extrato-centro percent ind8">' + '<input type="radio" name="valor_ind'+item.Id_indicadores+'" value="2"/>' + '</td>' +
											

									
													'</tr>')
											

										}
									}
					
					
								}
							}
				
						}
				
		
					}
				}				
			})

			$("#gcd-pesquisa").removeClass("gcd-loading");

		},
		beforeSend: function () {
			$(".loadeModal, .fundoModal").fadeIn();
		},
		complete: function () {
			$(".loadeModal, .fundoModal").fadeOut();
		}
	  

	})
	
            
}

Painel.js
$(function () {
	url = "http://192.168.0.42";
    $("#gcd-ct-extrato").addClass("gcd-loading");
	$("#gcd-ct-extrato_r").addClass("gcd-loading");

    $("#gcd-slider-ano").animate({
        marginLeft: '160px'
    });
    var ano = '2019';
	var cidade = 'Salvador'; 
    $(".detModal").hide();
    montatela(ano,cidade);
   
   

});

$(".fanterior").live("click", function(){
	$("#gcd-slider-ano").animate({
		marginLeft: '0px'
	},  500, function(){
		$(".fanterior").removeClass("gcd-fatura-inativa");
		$(".fatual").addClass("gcd-fatura-inativa");
		$("#gcd-ct-extrato").addClass("gcd-loading");
	});

	var ano = '2018';
	var cidade = 'Salvador';
	if (!$(".filtro_empresa").hasClass("gcd-fatura-inativa")) {
        cidade = 'Salvador'
    }
    else {
		  if (!$(".filtro_ope").hasClass("gcd-fatura-inativa")) {
				cidade = 'São Paulo'
		 }
         else {
			  if (!$(".filtro_cor").hasClass("gcd-fatura-inativa")) {
				cidade = 'Rio de Janeiro'
			  }
		}
	}
	
	$("#extrato-usuario").fadeOut(1000, function(){
	    $(this).remove();
	    montatela(ano,cidade);
	});
	
	return false;
});

$(".fatual").live("click", function(){
	$("#gcd-slider-ano").animate({
	    marginLeft: '160px'
	},  500, function(){
		$(".fatual").removeClass("gcd-fatura-inativa");
		$(".fanterior").addClass("gcd-fatura-inativa");
		$("#gcd-ct-extrato").addClass("gcd-loading");
	});

	var ano = '2019';
	var cidade = 'Salvador';
	if (!$(".filtro_empresa").hasClass("gcd-fatura-inativa")) {
        cidade = 'Salvador'
    }
    else {
		  if (!$(".filtro_ope").hasClass("gcd-fatura-inativa")) {
				cidade = 'São Paulo'
		 }
         else {
			  if (!$(".filtro_cor").hasClass("gcd-fatura-inativa")) {
				cidade = 'Rio de Janeiro'
			  }
		}
	}
	

	$("#extrato-usuario").fadeOut(1000, function () {
	    $(this).remove();
	    montatela(ano,cidade);
	});
	return false;
});

$(".filtro_empresa").live("click", function () {
    $("#gcd-slider-filtro").animate({
        marginLeft: '320px'
    }, 500, function () {
        $(".filtro_empresa").removeClass("gcd-fatura-inativa");
        $(".filtro_ope").addClass("gcd-fatura-inativa");
        $(".filtro_cor").addClass("gcd-fatura-inativa");
    });

    var ano;
    if (!$(".fanterior").hasClass("gcd-fatura-inativa")) {
        ano = '2018'
    };

    if (!$(".fatual").hasClass("gcd-fatura-inativa")) {
        ano = '2019'
    };
       
    $("#extrato-usuario").fadeOut(1000, function () {
	    $(this).remove();
	    montatela(ano,'Salvador');
	});
    return false;
});

$(".filtro_cor").live("click", function () {
    $("#gcd-slider-filtro").animate({
        marginLeft: '160px'
    }, 500, function () {
        $(".filtro_empresa").addClass("gcd-fatura-inativa");
        $(".filtro_ope").addClass("gcd-fatura-inativa");
        $(".filtro_cor").removeClass("gcd-fatura-inativa");
    });

    var ano;
    if (!$(".fanterior").hasClass("gcd-fatura-inativa")) {
        ano = '2018'
    };

    if (!$(".fatual").hasClass("gcd-fatura-inativa")) {
        ano = '2019'
    };
       
    $("#extrato-usuario").fadeOut(1000, function () {
	    $(this).remove();
	    montatela(ano,'São Paulo');
	});
    return false;
});

$(".filtro_ope").live("click", function () {
    $("#gcd-slider-filtro").animate({
        marginLeft: '0px'
    }, 500, function () {
        $(".filtro_empresa").addClass("gcd-fatura-inativa");
        $(".filtro_ope").removeClass("gcd-fatura-inativa");
        $(".filtro_cor").addClass("gcd-fatura-inativa");
    });

    var ano;
    if (!$(".fanterior").hasClass("gcd-fatura-inativa")) {
        ano = '2018'
    };

    if (!$(".fatual").hasClass("gcd-fatura-inativa")) {
        ano = '2019'
    };
       
    $("#extrato-usuario").fadeOut(1000, function () {
	    $(this).remove();
	    montatela(ano,'OPE');
	});
    return false;
});






$(".detFechar").live("click", function () {
    $(".detModal, .fundoModal").fadeOut();
})



function number_format(number, decimals, dec_point, thousands_sep) {
    var n = number, c = isNaN(decimals = Math.abs(decimals)) ? 2 : decimals;
    var d = dec_point == undefined ? "," : dec_point;
    var t = thousands_sep == undefined ? "." : thousands_sep, s = n < 0 ? "-" : "";
    var i = parseInt(n = Math.abs(+n || 0).toFixed(2)) + "", j = (j = i.length) > 3 ? j % 3 : 0;
    return s + (j ? i.substr(0, j) + t : "") + i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + t) + (c ? d + Math.abs(n - i).toFixed(2).slice(2) : "");
}




function montatela(ano,un) {


	var urlp = url+"/mvc/?pagina=painel&metodo=consulta";
	                          
	$.ajax({
		url:        urlp,
		dataType:   "json", 
		success: function (data) {

			
			$("#gcd-ct-extrato").append(
				'<table id="extrato-usuario" class="tablesorter">' +
					'<thead>' +
							'<tr>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">DOMINIO</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">ICD</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">SUBDOMINIO</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">IMSD</th>' +
								'<th class="gcd-extrato-centro icd">TOTAL</th>' +
								'<th class="gcd-extrato-centro icd">NA</th>' +
								'<th class="gcd-extrato-centro icd">PA</th>' +
								'<th class="gcd-extrato-centro icd">TA</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">PSD</th>' +
								'<th class="gcd-extrato-centro icd" rowspan="2">PD</th>' +
					  
							'</tr>' +
							'<tr>' +
								
								'<th class="gcd-extrato-centro icd">INDICADORES ISO</th>' +
								'<th class="gcd-extrato-centro icd">0</th>' +
								'<th class="gcd-extrato-centro icd">1</th>' +
								'<th class="gcd-extrato-centro icd">2</th>' +
								
					  
							'</tr>' +
							
						'</thead>'+
						'<tbody>' +
						'</tbody>' +
							'</table>'


					);

		 
			$.each(data, function (i, item) {
									   
				
				if ( item.ordem == "1" || item.ordem == "2")
				{
					if (item.ordem == "1"){
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
							'<td class="ind3" rowspan="2">' + item.dominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind3" rowspan="2">' + number_format(item.icd, 1, ',', '.') + '%</td>' +
							'<td class="ind3">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3" rowspan="2">' + number_format(item.pd, 0, ',', '.') + '</td>' +
							'</tr>')
					}
					else{
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
							'<td class="ind3">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind3">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							

					'</tr>')
					}
				}
				if ( item.ordem == "3")
				{
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
							'<td class="ind4">' + item.dominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + number_format(item.icd, 1, ',', '.') + '%</td>' +
							'<td class="ind4">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind4">' + number_format(item.pd, 0, ',', '.') + '</td>' +

					'</tr>')
				}
				if ( item.ordem == "4")
				{
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
							'<td class="ind5">' + item.dominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + number_format(item.icd, 1, ',', '.') + '%</td>' +
							'<td class="ind5">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind5">' + number_format(item.pd, 0, ',', '.') + '</td>' +

					'</tr>')
				}
				if ( item.ordem == "5" || item.ordem == "6")
				{
					if (item.ordem == "5"){
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
							'<td class="ind6" rowspan="2">' + item.dominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind6" rowspan="2">' + number_format(item.icd, 1, ',', '.') + '%</td>' +
							'<td class="ind6">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6" rowspan="2">' + number_format(item.pd, 0, ',', '.') + '</td>' +

					'</tr>')
					}
					else{
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
							
							'<td class="ind6">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind6">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							

					'</tr>')
					}
				}
				
				if ( item.ordem == "7" || item.ordem == "8" || item.ordem == "9" || item.ordem == "10" || item.ordem == "11" || item.ordem == "12" || item.ordem == "13")
				{
					if (item.ordem == "7"){
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
							'<td class="ind7" rowspan="7">' + item.dominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind7" rowspan="7">' + number_format(item.icd, 1, ',', '.') + '%</td>' +
							'<td class="ind7">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7" rowspan="7">' + number_format(item.pd, 0, ',', '.') + '</td>' +

					'</tr>')
					}
					else{
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
						
							'<td class="ind7">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind7">' + number_format(item.psd, 0, ',', '.') + '</td>' +
					

					'</tr>')
					}
				}
				if ( item.ordem == "14" || item.ordem == "15" || item.ordem == "16" || item.ordem == "17" || item.ordem == "18" || item.ordem == "19")
				{
					if(item.ordem =="14"){
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
							'<td class="ind8" rowspan="6">' + item.dominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind8" rowspan="6">' + number_format(item.icd, 1, ',', '.') + '%</td>' +
							'<td class="ind8">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8" rowspan="6">' + number_format(item.pd, 0, ',', '.') + '</td>' +

					'</tr>')
					}
					else{
					$("#extrato-usuario tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.dominio + '">' +
				   
							
							'<td class="ind8">' + item.subdominio + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.imsd, 1, ',', '.') + '%</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.indicadores, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.na, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.pa, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.ta, 0, ',', '.') + '</td>' +
							'<td class="gcd-extrato-centro percent ind8">' + number_format(item.psd, 0, ',', '.') + '</td>' +
							

					'</tr>')
					}
				}
				
		
							
			})

			$("#gcd-ct-extrato").removeClass("gcd-loading");

		},
		beforeSend: function () {
			$(".loadeModal, .fundoModal").fadeIn();
		},
		complete: function () {
			$(".loadeModal, .fundoModal").fadeOut();
		}
	  

	})
	
            
}

Relatorio.js
$(function () {
	url = "http://192.168.0.42";
    $("#gcd-ct-extrato").addClass("gcd-loading");
	$("#gcd-ct-extrato_r").addClass("gcd-loading");

    
    $(".detModal").hide();
    montatela();
   
   

});



$(".detFechar").live("click", function () {
    $(".detModal, .fundoModal").fadeOut();
})



function number_format(number, decimals, dec_point, thousands_sep) {
    var n = number, c = isNaN(decimals = Math.abs(decimals)) ? 2 : decimals;
    var d = dec_point == undefined ? "," : dec_point;
    var t = thousands_sep == undefined ? "." : thousands_sep, s = n < 0 ? "-" : "";
    var i = parseInt(n = Math.abs(+n || 0).toFixed(2)) + "", j = (j = i.length) > 3 ? j % 3 : 0;
    return s + (j ? i.substr(0, j) + t : "") + i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + t) + (c ? d + Math.abs(n - i).toFixed(2).slice(2) : "");
}


function _GETParametro(name) {
    var url = window.location.search.replace("?", "");
    var itens = url.split("&");

    for (n in itens) {
        if (itens[n].match(name)) {
            return decodeURIComponent(itens[n].replace(name + "=", ""));
        }
    }
    return null;
}

function montatela() {



	var urlpr = url+"/mvc/?pagina=painel&metodo=consulta_resumo";
	$.ajax({
		url:        urlpr,
		dataType:   "json", 
		success: function (data) {
	
			$("#gcd-ct-extrato_r").append(
				'<table id="extrato-usuario_r" class="tablesorter">' +
					'<thead>' +
							
						'</thead>'+
						'<tbody>' +
						'</tbody>' +
							'</table>'


					);

		 
			$.each(data, function (i, item) {
									   
				
				if ( item.ordem == "1")
				{
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + 'CIDADE' + '">' +
				   
							'<td class="icd">' + 'CIDADE' + '</td>' +
							'<td class="gcd-extrato-centro int icd">' + 'ANO' + '</td>' +

					'</tr>')
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.cidade + '">' +
				   
							'<td class="totais">' + item.cidade + '</td>' +
							'<td class="gcd-extrato-centro int valor">' + item.ano + '</td>' +

					'</tr>')
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + 'CIDADE' + '">' +
				   
							'<td class="icd">' + 'TOTAIS' + '</td>' +
							'<td class="gcd-extrato-centro int icd">' + '' + '</td>' +

					'</tr>')
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.descr + '">' +
				   
							'<td class="totais">' + item.descr + '</td>' +
							'<td class="gcd-extrato-centro percent valor">' + number_format(item.valor, 0, ',', '.') + '</td>' +

					'</tr>')
				}
				if ( item.ordem == "2")
				{
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.descr + '">' +
				   
							'<td class="totais">' + item.descr + '</td>' +
							'<td class="gcd-extrato-centro percent valor">' + number_format(item.valor, 1, ',', '.') + '%</td>' +
						'</tr>' +
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + 'icd' + '">' +
				   
							'<td class="icd">' + 'Índice De Conformidade Por Domínio' + '</td>' +
							'<td class="icd">' + 'ICD' + '</td>' +
							
					'</tr>')
				}
				if ( item.ordem == "3")
					
				{
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.descr + '">' +
				   
							'<td class="ind3">' + item.descr + '</td>' +
							'<td class="gcd-extrato-centro decimal valor">' + number_format(item.valor, 1, ',', '.') + '%</td>' +

					'</tr>')
				}
				if ( item.ordem == "4")
				{
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.descr + '">' +
				   
							'<td class="ind4">' + item.descr + '</td>' +
							'<td class="gcd-extrato-centro decimal valor">' + number_format(item.valor, 1, ',', '.') + '%</td>' +

					'</tr>')
				}
				if ( item.ordem == "5")
				{
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.descr + '">' +
				   
							'<td class="ind5">' + item.descr + '</td>' +
							'<td class="gcd-extrato-centro decimal valor">' + number_format(item.valor, 1, ',', '.') + '%</td>' +

					'</tr>')
				}
				if ( item.ordem == "6")
				{
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.descr + '">' +
				   
							'<td class="ind6">' + item.descr + '</td>' +
							'<td class="gcd-extrato-centro decimal valor">' + number_format(item.valor, 1, ',', '.') + '%</td>' +

					'</tr>')
				}
				if ( item.ordem == "7")
				{
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.descr + '">' +
				   
							'<td class="ind7">' + item.descr + '</td>' +
							'<td class="gcd-extrato-centro decimal valor">' + number_format(item.valor, 1, ',', '.') + '%</td>' +

					'</tr>')
				}
				if ( item.ordem == "8")
				{
					$("#extrato-usuario_r tbody").append(
						'<tr class="' + 'gcd-extrato-tr-credito' + '" title="' + item.descr + '">' +
				   
							'<td class="ind8">' + item.descr + '</td>' +
							'<td class="gcd-extrato-centro decimal valor">' + number_format(item.valor, 1, ',', '.') + '%</td>' +

					'</tr>')
				}
		
							
			})

			$("#gcd-ct-extrato_r").removeClass("gcd-loading");

		},
		beforeSend: function () {
			$(".loadeModal, .fundoModal").fadeIn();
		},
		complete: function () {
			$(".loadeModal, .fundoModal").fadeOut();
		}
	  

	})
            
}
























