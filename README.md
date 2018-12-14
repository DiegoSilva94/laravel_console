# laravel_console
Generate validation code command
```php
Artisan::command('validate {table}', function ($table) {
    function getValidationsFromTable($table)
    {
        if($table == null)
            return '';
        $retorno = '';
        $columns = DB::select('SELECT * FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = ? AND TABLE_NAME = ?', [env('DB_DATABASE'), $table]);
        foreach($columns as $column)
        {
            $condicoes = [];
            if($column->COLUMN_NAME == 'id' || $column->COLUMN_NAME == 'updated_at' || $column->COLUMN_NAME == 'created_at')
                continue;
            if($column->IS_NULLABLE == 'NO')
                $condicoes[] = 'required';
            if($column->IS_NULLABLE != 'NO')
                $condicoes[] = 'nullable';
            $condicoes[] = getDataType(($column->DATA_TYPE != 'enum'? $column->DATA_TYPE : $column->COLUMN_TYPE));
            if($column->CHARACTER_MAXIMUM_LENGTH != null)
                $condicoes[] = 'max:' . $column->CHARACTER_MAXIMUM_LENGTH;
            if($column->COLUMN_KEY != "")
                $condicoes[] = getColumnKey($column->TABLE_NAME, $column->COLUMN_NAME);
            $retorno .= "\t\t".'\'' . $column->COLUMN_NAME . '\' => \'' . implode('|',$condicoes) . '\','."\n";
        }
        return substr($retorno,0,-2);
    }

    /**
     * @param $dataType
     * @return string
     */
    function getDataType($dataType)
    {
        $in = 'enum';
        $string     = [ 'char', 'varchar', 'tinytext', 'text', 'mediumtext', 'longtext' ];
        $data       = [ 'date', 'datetime', 'timestamp', 'time', 'year' ];
        $integer    = [ 'tinyint', 'smallint', 'mediumint', 'int', 'bigint' ];
        $numeric    = [ 'float', 'double', 'decimal' ];
        if(strstr($dataType, $in))
            return 'in:' . str_replace('\',\'',',',str_replace('\')','',str_replace('enum(\'','',$dataType)));
        if(in_array($dataType, $string))
            return 'string';
        if(in_array($dataType, $data))
            return 'data';
        if(in_array($dataType, $integer))
            return 'integer';
        if(in_array($dataType, $numeric))
            return 'numeric';
        return '';
    }

    /**
     * @param $table
     * @param $column
     * @return string
     */
    function getColumnKey($table, $column)
    {
        $keysColumn = DB::select('SELECT * FROM information_schema.KEY_COLUMN_USAGE WHERE TABLE_SCHEMA = ? AND TABLE_NAME = ? AND COLUMN_NAME = ?', [env('DB_DATABASE'), $table, $column]);
        $retorno = [];
        foreach($keysColumn as $keyColumn)
        {
            if(strstr($keyColumn->CONSTRAINT_NAME, 'foreign'))
                $retorno[] = 'exists:' . $keyColumn->REFERENCED_TABLE_NAME . ',' . $keyColumn->REFERENCED_COLUMN_NAME;
            if(strstr($keyColumn->CONSTRAINT_NAME, 'unique'))
                $retorno[] = 'unique:' . $keyColumn->TABLE_NAME . ',' . $keyColumn->COLUMN_NAME;
        }
        return implode('|', $retorno);
    }
    $this->info(getValidationsFromTable($table));
})->describe('Display an validate code');
```
Command to generate relationship code
```php
Artisan::command('model {table}', function ($table) {

    /**
     * @param $table
     * @return string
     */
    function getFillableFromTable($table)
    {
        if($table == null)
            return '';
        $retorno = '';
        $columns = DB::select('SELECT * FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = ? AND TABLE_NAME = ?', [env('DB_DATABASE'), $table]);
        foreach($columns as $column)
        {
            if($column->COLUMN_NAME == 'id' || $column->COLUMN_NAME == 'updated_at' || $column->COLUMN_NAME == 'created_at')
                continue;
            $retorno .= "\t\t".'\'' . $column->COLUMN_NAME . '\','."\n";
        }
        return substr($retorno,0,-2);
    }
    $this->info(getFillableFromTable($table));
})->describe('Display an Relationships code');
```
Database create command
```php
Artisan::command('migrate:database', function () {
    extract(config('database.connections')[config('database.default')]);
    $this->info("creating database: {$database} in {$host}");
    try {
        $pdo = new \PDO("{$driver}:host={$host}", $username, $password);
        $pdo->query("CREATE DATABASE IF NOT EXISTS {$database} CHARACTER SET {$charset} COLLATE  {$collation}");
        $this->info('created successfully');
    } catch (Exception $e) {
        $this->error('failure: ' . $e->getMessage());
    }
});
```
