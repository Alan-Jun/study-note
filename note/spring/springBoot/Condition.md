@ConditionalOnBean         //	当给定的在bean存在时,则实例化当前Bean 

@ConditionalOnMissingBean  //	当给定的在bean不存在时,则实例化当前Bean 

@ConditionalOnClass        //	当给定的类名在类路径上存在，则实例化当前Bean 

@ConditionalOnMissingClass //	当给定的类名在类路径上不存在，则实例化当前Bean