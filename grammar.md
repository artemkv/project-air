token:=<comment>|<endpoint>

comment:=//<text>

text:=?

endpont:=
<verb> "<url>" =>
	from <source_type> [<source>]
	<return>|<insert>|<update>|<delete>
	
verb:=GET|POST|PUT|DELETE

url:=text

source_type:=table|body

source:=text

return:=<return_single>|<return_many>

return_single:=
	return by <id_column>:
		<columns>

return_many:=
	return many [with <options>]
		<columns>

options:=paging|sorting|filter

columns:=
	<column>[,<columns>]