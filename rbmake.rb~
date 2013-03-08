def initalize
  filename = 'Makefile'
  @lineArr = Array.new
  @Makefile = File.new( filename, 'r' )
  @dag = Hash.new  #hash of all the dependents to their dependencies
  @updateCommands = Hash.new # hash of all the dependents to their update 
                             # commands
  @makeProcess = Hash.new # hash of all the files in the make file to a 
                          # string representing processed or not
  @fileModTimes = Hash.new # hash of all files to their modification times
  @maccros = Hash.new
  @target = ARGV[0]
end

def parseFile ( line )
  line.gsub!(/#.*/, '' )
  line.gsub!(/\A\s+\z/, '' )
  line.chomp! 
  if( line != ''  )
    @lineArr << line
  end 
end 

def parseArr
  arrPos = 0
  #find the line containing 'initalize:' should be the first line
  while( @lineArr[arrPos] != 'initialize:')
    arrPos +=1
  end
  arrPos +=1 #increment past "initialize:" line
  # System commands for initalization. Executed through bash
  while( @lineArr[arrPos].include? "\t" )
    cmd = @lineArr[arrPos].strip
    system( cmd ) 
    arrPos += 1
  end
  # arrPos is now at location of  first non system commmand line
  while( @lineArr[arrPos].include?( "=" ) )
    tempArr = Array.new
    tempArr = @lineArr[arrPos].split("=")
    maccro = tempArr[0].strip
    substitute = tempArr[1].strip
    @maccros[maccro] = substitute
    arrPos += 1
  end
  while( arrPos < @lineArr.size )
    #while( @lineArr[arrPos].include?("$") )
      #this is where I would replace the maccros with their meanings
      # that I have already stored in a hash. However I do not know how
      # to capture regular expressions and I have run out of time 
    #end 
    # line containing a dependency. Split the line and hash the 
    # dependent to an array of its dependencies in @dag
    tempArr = Array.new
    tempArr = @lineArr[arrPos].split(":")
    dependent = tempArr[0].strip
    tempArr = tempArr[1].split
    @dag[dependent] = tempArr
    arrPos += 1
    # iterate over the file until a line without a tab has been found 
    # or the end of the array has been reached, grabbing each line
    # and storing it as a command. 
    cmdArr = Array.new
    while( arrPos < @lineArr.size && @lineArr[arrPos].include?( "\t" ) )  
      cmdArr << @lineArr[arrPos].strip
      arrPos += 1
    end
    @updateCommands[dependent] = cmdArr
  end
end

#return false if any of the dependency mod times are later then
# the mod time of fileName. True otherwise
def upToDate?( fileName )
   bool = true
   @dag[fileName].each do |element|
     if( (@fileModTimes[fileName] <=> @fileModTimes[element]) < 1 ) 
       bool = false
     end
   end
   return bool
end

def addItemToHashes( item )
  @makeProcess[item] = "unprocessed"
  if( File.exists?( item ) )
    @fileModTimes[item] = File.new(item, 'r').mtime
  else
    @fileModTimes[item] = Time.now
  end

end

def makeProcessHash
  @dag.each do |key, value|
    addItemToHashes( key )
    @dag[key].each do |element|
      addItemToHashes(element)
    end
  end
end


#returns a boolean representing id this file is up to date
def makePart( fileName )
  if( @makeProcess[fileName] == "processed" )#target already processed
    #puts fileName + " : " + @makeProcess[fileName]
    return true
  elsif( !@dag.key?(fileName) ) # target not on a dependency line
    if( !File.exists?( fileName ) )
      puts "Create file " + fileName
      # fix for part 4
      @fileModTimes[fileName] = Time.now
      #@makeProcess[fileName] = "processed"
    else
      @fileModTimes[fileName] = File.new(fileName, 'r').mtime 
      @makeProcess[fileName] = "processed"
    end
  else #fileName is listed on a dependency line
    @dag[fileName].each do |element|
      if( element != nil ) 
        makePart( element )
      end
    end
    # the file does not exist or it needs to be updated
    if( !File.exists?( fileName ) || !upToDate?(fileName ))
      if( @updateCommands.has_key?(fileName ) )
        @updateCommands[fileName].each do |element|
          system( element )
          exitNum = $?.exitstatus
          if ( exitNum != 0 )
            printf( "Command exit with status %i\n ", exitNum )
          end
        end
      else
        puts "Do not know how to make " + fileName
        exit(0)
      end
      @fileModTimes[fileName] = Time.now
      @makeProcess[fileName] = "processed"
    else
      @fileModTimes[fileName] = File.new( fileName, 'r' ).mtime
      @makeProcess[fileName] = "processed"
    end
  end
end

# recursively prints the specified file and all of its dependencies 
# benith it indented.
def printDependencies( dependent, hash, indent )
  puts indent + dependent
  if( hash.key?( dependent ) )
    hash[dependent].each do |element|
      printDependencies( element, hash, indent + '  ' )
    end
  end  
end

initalize
@Makefile.each do |line|
  parseFile( line )
end
parseArr
makeProcessHash
makePart( @target )
