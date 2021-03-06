# Demo for strctured programming blog
```
#include <complex>
template <class Snd>
using sender_result_t = typename Snd::result_t;
struct None{};
template<typename T,typename Rcvr>
struct just_op{
    T val;
    Rcvr rcvr;
   
    void start(){
        set_value(rcvr,val);
    }
};
template<typename T>
struct just{
   using result_t = T;
   T val;
   just(T v):val(std::move(v)){}
   template<typename Rcvr>
   friend auto connect(just self,Rcvr rcvr){
       return just_op<T,Rcvr>{self.val,rcvr};
   }
};
template<typename Rcvr,typename Func>
struct arith_reciever{
    Rcvr rcvr;
    Func func;
    friend void set_value(arith_reciever& self,auto val){
        set_value(self.rcvr,self.func(val));
    }
};
template<typename PrevSend, typename Func>
struct arith_sender{
    using prev_result_t = sender_result_t<PrevSend>;
    using result_t = std::invoke_result_t<Func, prev_result_t>;
    PrevSend prev_send;
    Func fun;
    template<typename Rcvr>
    friend auto connect(arith_sender self,Rcvr rcvr){
        return connect(self.prev_send,arith_reciever<Rcvr,Func>{rcvr,self.fun});
    }
};
template<typename T,typename Func>
arith_sender(T,Func)->arith_sender<T,Func>;

template<typename T>
struct eval_reciever{
    T& val;
    friend void set_value(eval_reciever self,auto v){
        self.val=v;
    }
};
template<typename Send>
sender_result_t<Send> evaluator(Send send){
    using T =typename Send::result_t;
    T v;
    auto op=connect(send,eval_reciever<T>{v});
    op.start();
    return v;
}
auto add_to(auto val,auto sender){
    return arith_sender{sender,[val=std::move(val)](auto v){return val+v;}};
}
auto mult_with(auto val,auto sender){
    return arith_sender{sender,[val=std::move(val)](auto v){return val*v;}};
}
auto div_by(auto val,auto sender){
    return arith_sender{sender,[val=std::move(val)](auto v){return val/v;}};
}
auto operator|(auto send,auto func){
    return arith_sender{send,func};
}

using namespace std;
auto add(auto a, auto b){
    return a+b;
}
auto mult(auto a, auto b){
    return a+b;
}
auto div(auto a, auto b){
    return a+b;
}

int  main(){
    auto res = add(complex(8,6),
                 mult(complex(5,7),
                     div(complex(2,3),complex(3,5))));
    
    auto res2 = add(std::string("Hello"),std::string("world"));
    auto my_complex_algo=[](auto send){
        return div_by(complex(13,17),
                    mult_with(complex(1,2),
                        add_to(complex(5,6),
                            send)));
    };
    auto eval = my_complex_algo(just(complex(1,3)));                    
    auto v= evaluator(eval);
    printf("%d,i%d\n",v.real(),v.imag());

    auto eval2= just(complex(1,3))
                    |[](auto v){return complex(5,6)+v;}
                    |[](auto v){return complex(1,2)*v;}
                    |[](auto v){return complex(13,17)/v;}
                    ;
    auto v2= evaluator(eval2);
    printf("%d,i%d\n",v2.real(),v2.imag());

    auto eval3= just(std::string("hello"))
                    |[](auto v){return v+"world";}
                    |[](auto v){return  "{"+v+"}";}
                    |[](auto v){return  "<B>"+v+"</B>";};

     printf("%s", evaluator(eval3).data());
    
}
```
