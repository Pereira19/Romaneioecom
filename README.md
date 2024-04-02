<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Romaneio Ecommerce</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/normalize/8.0.1/normalize.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <style>
        /* Estilos CSS */
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            margin: 0;
            padding: 0;
        }
        .container {
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            background-color: #fff;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            text-align: center;
            color: #333;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        .deleteBtn {
            background-color: #e74c3c;
            color: white;
            border: none;
            padding: 6px 12px;
            border-radius: 4px;
            cursor: pointer;
        }
        .deleteBtn:hover {
            background-color: #c0392b;
        }
        .deleteAllBtn {
            background-color: #e74c3c;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 4px;
            cursor: pointer;
            margin-bottom: 20px;
        }
        .deleteAllBtn:hover {
            background-color: #c0392b;
        }
        .addRowBtn {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            margin-bottom: 20px;
        }
        .addRowBtn:hover {
            background-color: #2980b9;
        }
        #totalItems {
            display: block;
            margin-top: 20px;
            text-align: right;
            font-size: 18px;
        }
        .controls {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 20px;
        }
        .searchInput {
            flex: 1;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
            width: calc(100% - 140px); /* Largura do campo de busca */
        }
        .searchBtn {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 4px;
            cursor: pointer;
            margin-left: 10px; /* Espaçamento entre o campo de busca e o botão de pesquisa */
        }
        .searchBtn:hover {
            background-color: #2980b9;
        }
        #txtDateFilter {
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
            width: 150px; /* Largura do campo de filtro de data */
        }
        #clearDateFilterBtn {
            background-color: #e74c3c;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 4px;
            cursor: pointer;
        }
        #clearDateFilterBtn:hover {
            background-color: #c0392b;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Romaneio Ecommerce</h1>
        <div class="controls">
            <div style="display: flex;">
                <input type="text" class="searchInput" placeholder="Buscar...">
                <button class="searchBtn" id="searchBtn">Pesquisar</button>
            </div>
            <input type="text" id="txtDateFilter" placeholder="Filtrar por data (dd/mm/aaaa)">
            <button id="clearDateFilterBtn">Limpar Filtro</button>
        </div>
        <table id="tblData">
            <thead>
                <tr>
                    <th>Cliente</th>
                    <th>Produto</th>
                    <th>Quantidade</th>
                    <th>Valor</th>
                    <th>Data</th>
                    <th>Ações</th>
                </tr>
            </thead>
            <tbody>
                <!-- Linhas de dados serão adicionadas aqui -->
            </tbody>
        </table>
        <button class="addRowBtn" id="addRowBtn">Adicionar Linha</button>
        <button class="deleteAllBtn">Excluir Todas as Linhas</button>
        <span id="totalItems">Total: <span id="totalCount">0</span> itens</span>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script>
        $(function() {
            // Adiciona linhas salvas ao carregar a página
            var rows = JSON.parse(localStorage.getItem('rows')) || [];
            if (rows.length > 0) {
                rows.forEach(function(row) {
                    addRow(row);
                });
                updateTotalItems(rows);
            }

            $('#addRowBtn').click(function() {
                addRow([]);
                updateLocalStorage();
            });

            $('#tblData').on('click', '.deleteBtn', function() {
                $(this).closest('tr').remove();
                updateTotalItems(getRowsData());
                updateLocalStorage();
            });

            $('#tblData').on('input', '.editable', function() {
                updateLocalStorage();
            });

            $('#searchBtn').click(function() {
                searchTable();
            });

            $("#txtDateFilter").datepicker({
                dateFormat: 'dd/mm/yy',
                onSelect: function(dateText) {
                    filterByDate(dateText);
                }
            });

            $('#clearDateFilterBtn').click(function() {
                $('#txtDateFilter').val('');
                filterByDate('');
            });

            $('.deleteAllBtn').click(function() {
                $('#tblData tbody').empty();
                updateTotalItems([]);
                updateLocalStorage();
            });

            $('#tblData').on('paste', function(e) {
                e.preventDefault();
                var text = (e.originalEvent || e).clipboardData.getData('text/plain');
                var rows = text.split('\n');
                var tbody = $('#tblData tbody');

                rows.forEach(function(rowData) {
                    var cells = rowData.split('\t');
                    var tr = $('<tr>');

                    cells.forEach(function(cellData) {
                        var td = $('<td contenteditable="true" class="editable"></td>');
                        td.text(cellData.trim());
                        tr.append(td);
                    });

                    tr.append('<td><button class="deleteBtn">Excluir</button></td>');
                    tbody.append(tr);
                });

                updateLocalStorage();
                updateTotalItems(getRowsData());
            });

            function addRow(data) {
                var tbody = $('#tblData tbody');
                var tr = $('<tr>');
                tr.append('<td contenteditable="true" class="editable">' + (data[0] || '') + '</td>');
                tr.append('<td contenteditable="true" class="editable">' + (data[1] || '') + '</td>');
                tr.append('<td contenteditable="true" class="editable">' + (data[2] || '') + '</td>');
                tr.append('<td contenteditable="true" class="editable">' + (data[3] || '') + '</td>');
                tr.append('<td contenteditable="true" class="editable">' + (data[4] || '') + '</td>');
                tr.append('<td><button class="deleteBtn">Excluir</button></td>');
                tbody.append(tr);
            }

            function updateLocalStorage() {
                var rowsData = getRowsData();
                localStorage.setItem('rows', JSON.stringify(rowsData));
            }

            function getRowsData() {
                var rowsData = [];
                $('#tblData tbody tr').each(function() {
                    var rowData = [];
                    $(this).find('td.editable').each(function() {
                        rowData.push($(this).text());
                    });
                    rowsData.push(rowData);
                });
                return rowsData;
            }

            function updateTotalItems(rowsData) {
                var totalItems = rowsData.length;
                $('#totalCount').text(totalItems);
            }

            function searchTable() {
                var searchText = $('.searchInput').val().toLowerCase();
                $('#tblData tbody tr').each(function() {
                    var cells = $(this).find('td');
                    var found = false;
                    cells.each(function() {
                        if ($(this).text().toLowerCase().includes(searchText)) {
                            found = true;
                        }
                    });
                    $(this).toggle(found);
                });
            }

            function filterByDate(dateText) {
                var rows = $('#tblData tbody tr');
                rows.each(function() {
                    var dateCell = $(this).find('td').eq(4); // Quinta coluna contém a data
                    var date = dateCell.text().trim();
                    if (date === dateText || dateText === '') {
                        $(this).show();
                    } else {
                        $(this).hide();
                    }
                });
            }
        });
    </script>
</body>
</html>
