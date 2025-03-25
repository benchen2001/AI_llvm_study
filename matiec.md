/* The base class of all symbols */
class symbol_c {

  public:
    /* WARNING: only use this method for debugging purposes!! */
    virtual const char *absyntax_cname(void) {return "symbol_c";};

    /*
     * Annotations produced during stage 1_2
     */    
    /* Points to the parent symbol in the AST, i.e. the symbol in the AST that will contain the current symbol */
    symbol_c *parent;
    /* Some symbols may not be tokens, but may be clearly identified by a token.
     * For e.g., a FUNCTION declaration is not itself a token, but may be clearly identified by the
     * token_c object that contains it's name. Another example is an element in a STRUCT declaration,
     * where the structure_element_declaration_c is not itself a token, but can be clearly identified
     * by the structure_element_name
     * To make it easier to find these tokens from the top level object, we will have the stage1_2 populate this
     * token_c *token wherever it makes sense.
     * NOTE: This was a late addition to the AST. Not all objects may be currently so populated.
     *       If you need this please make sure the bison code is populating it correctly for your use case.
     */
    token_c  *token;
    
    /* Line number for the purposes of error checking.  */
    int first_line;
    int first_column;
    const char *first_file;  /* filename referenced by first line/column */
    long int first_order;    /* relative order in which it is read by lexcial analyser */
    int last_line;
    int last_column;
    const char *last_file;  /* filename referenced by last line/column */
    long int last_order;    /* relative order in which it is read by lexcial analyser */

    int source;  /* 0: symbol from libraries, 1: symbol from user code */

    /*
     * Annotations produced during stage 3
     */    
    /*** Data type analysis ***/
    std::vector <symbol_c *> candidate_datatypes; /* All possible data types the expression/literal/etc. may take. Filled in stage3 by fill_candidate_datatypes_c class */
    /* Data type of the expression/literal/etc. Filled in stage3 by narrow_candidate_datatypes_c 
     * If set to NULL, it means it has not yet been evaluated.
     * If it points to an object of type invalid_type_name_c, it means it is invalid.
     * Otherwise, it points to an object of the apropriate data type (e.g. int_type_name_c, bool_type_name_c, ...)
     */
    symbol_c *datatype;
    /* The POU in which the symbolic variable (or structured variable, or array variable, or located variable, - any more?)
     * was declared. This will point to a Configuration, Resource, Program, FB, or Function.
     * This is set in stage 3 by the datatype analyser algorithm (fill/narrow) for the symbols:
     *  symbolic_variable_c, array_variable_c, structured_variable_c
     */
    symbol_c *scope;    

    /*** constant folding ***/
    /* If the symbol has a constant numerical value, this will be set to that value by constant_folding_c */
    const_value_c const_value;
    
    /*** Enumeration datatype checking ***/    
    /* Not all symbols will contain the following anotations, which is why they are not declared here in symbol_c
     * They will be declared only inside the symbols that require them (have a look at absyntax.def)
     */
    typedef std::multimap<std::string, symbol_c *, nocasecmp_c> enumvalue_symtable_t;
    
    /*
     * Annotations produced during stage 4
     */
    /* Since we support several distinct stage_4 implementations, having explicit entries for each
     * possible use would quickly get out of hand.
     * We therefore simply add a map, that each stage 4 may use for all its needs.
     */
    typedef std::map<std::string, symbol_c *> anotations_map_t;
    anotations_map_t anotations_map;
    

  public:
    /* default constructor */
    symbol_c(int fl = 0, int fc = 0, const char *ffile = NULL /* filename */, long int forder=0, /* order in which it is read by lexcial analyser */
             int ll = 0, int lc = 0, const char *lfile = NULL /* filename */, long int lorder=0  /* order in which it is read by lexcial analyser */
            );

    /* default destructor */
    /* must be virtual so compiler does not complain... */ 
    virtual ~symbol_c(void) {return;};

    virtual void *accept(visitor_c &visitor) {return NULL;};
};
