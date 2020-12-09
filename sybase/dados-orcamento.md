#### Verifica todas as despesas do PPA

```sql
select tipo_registro, 
                 sistema,
                 chave_dsk1,
                 chave_dsk2,
                 chave_dsk3,
                 chave_dsk4,
                 idPpa,
                 idNatureza,
                 idOrganograma,
                 min(i_rec_ppa) as numero,
                 sum(metaValorTotal) as metaValorTotal,
                 sum(metaValorAno1)  as metaValorAno1,
                 sum(metaValorAno2)  as metaValorAno2,
                 sum(metaValorAno3)  as metaValorAno3,
                 sum(metaValorAno4)  as metaValorAno4, 
                 list(distinct i_rec_ppa order by i_rec_ppa) as listaReceitas
            from (select 'receitasPpa'                           as tipo_registro, 
                         333                                     as sistema,
                         chaveDskNatureza                        as chave_dsk1,
                         hist_rec_ppa.i_entidades                as chave_dsk2,
                         isNull(hist_rec_ppa.i_organogramas, '') as chave_dsk3,
                         ppas.i_ppas                             as chave_dsk4,
                         hist_rec_ppa.i_rec_ppa, 
                         isnull(metas_arrecad_ppa.valor, 0) as metaValorTotal,
                         isnull(bethadba.dbf_get_id_gerado(sistema, 'ppas', ppas.i_ppas), '')            as idPpa, 
                         isnull(bethadba.dbf_get_id_gerado(1, 'naturezaReceitas', chaveDskNatureza), '') as idNatureza, 
                         isnull(desmembramento_arrecad.i_rubricas_principal,i_rubricas)                  as rubrica, 
                         isnull(bethadba.dbf_get_id_gerado(1, 'organogramas', chave_dsk3, ppas.i_ppas), '') as idOrganograma, 
                         left(if left(rubrica, 1) = '4' then substr(rubrica, 2, length(rubrica)-1) else rubrica endif, 14)     as chaveDskNatureza, 
                         isnull(metas_arrecad_anuais_ppa.vlr_ano1, 0) as metaValorAno1, 
                         isnull(metas_arrecad_anuais_ppa.vlr_ano2, 0) as metaValorAno2, 
                         isnull(metas_arrecad_anuais_ppa.vlr_ano3, 0) as metaValorAno3, 
                         isnull(metas_arrecad_anuais_ppa.vlr_ano4, 0) as metaValorAno4, 
                         isnull(desmembramento_arrecad.percentual,metas_arrecad_ppa.percentual) as percentual 
                    from bethadba.ppas 
                         join bethadba.rec_ppa on(rec_ppa.i_ppas = ppas.i_ppas) 
                         join bethadba.hist_rec_ppa on (rec_ppa.i_ppas = hist_rec_ppa.i_ppas and
                                                        rec_ppa.i_rec_ppa = hist_rec_ppa.i_rec_ppa) 
                         join sapo.rubricas on (rubricas.i_planos_contas = hist_rec_ppa.i_planos_contas and 
                                                rubricas.i_contas        = hist_rec_ppa.i_contas and
                                                rubricas.portaria_rubr   = ppas.i_exercicios)
                         join bethadba.metas_arrecad_ppa on (metas_arrecad_ppa.i_ppas         = hist_rec_ppa.i_ppas and
                                                             metas_arrecad_ppa.i_rec_ppa      = hist_rec_ppa.i_rec_ppa and
                                                             metas_arrecad_ppa.i_dt_alteracao = hist_rec_ppa.i_dt_alteracao)
                         left join bethadba.metas_arrecad_anuais_ppa on (metas_arrecad_anuais_ppa.i_ppas              = metas_arrecad_ppa.i_ppas and
                                                                         metas_arrecad_anuais_ppa.i_rec_ppa           = metas_arrecad_ppa.i_rec_ppa and
                                                                         metas_arrecad_anuais_ppa.i_dt_alteracao      = metas_arrecad_ppa.i_dt_alteracao and
                                                                         metas_arrecad_anuais_ppa.i_metas_arrecad_ppa = metas_arrecad_ppa.i_metas_arrecad_ppa) 
                         left outer join sapo.desmembramento_arrecad on (desmembramento_arrecad.portaria_rubr       = rubricas.portaria_rubr and 
                                                                         desmembramento_arrecad.i_entidades         = hist_rec_ppa.i_entidades and 
                                                                         desmembramento_arrecad.i_rubricas_auxiliar = rubricas.i_rubricas) 
                   where 2019 between ppas.i_exercicios and ppas.exerc_fin 
                     AND ppas.dt_aprovacao is not null
                     AND metaValorTotal > 0 
                     AND rubricas.i_rubricas not like '9%' 
                     AND rubricas.i_rubricas not like '49%') as tab2 
           where bethadba.dbf_get_situacao_registro(sistema,tipo_registro,chave_dsk1,chave_dsk2,chave_dsk3,chave_dsk4) in(5,3) 
           group by tipo_registro, sistema, chave_dsk1, chave_dsk2, chave_dsk3, chave_dsk4, idNatureza, idEntidade, idOrganograma, idPpa
           order by chave_dsk1, chave_dsk2, chave_dsk3, chave_dsk4
```

#### Verifica todas as receitas do PPA

```sql
select 
  entidade = i_entidades,
  organograma = chaveDsk1Organogramas,
  naturezaDespesa = chaveDskNaturezaDespesa,
  funcao = chaveDskFuncoes,
  subfuncao = chaveDskSubFuncoes ,
  programa = chaveDsk1Programas,
  acao = chaveDsk1Acoes,
  exercio = exercicio,
  sum(metaValorTotal) as metaValorTotal,
  sum(metaValorAno1) as metaValorAno1,
  sum(metaValorAno2) as metaValorAno2,
  sum(metaValorAno3) as metaValorAno3,
  sum(metaValorAno4) as metaValorAno4   
from (select chaveDsk1Organogramas = hist_planej_ppa.i_organogramas,
             chaveDskNaturezaDespesa = left(contas.mascara, 14),
             chaveDsk1Programas = cast(planej_ppa.i_programas as integer),
             chaveDsk1Acoes = planej_ppa.i_acoes,
             chaveDskFuncoes = hist_planej_ppa.i_funcoes,
             chaveDskSubFuncoes = hist_planej_ppa.i_subfuncoes,                         
             exercicio = 2019,
             hist_planej_ppa.i_entidades,
             chaveDskPpa  = ppas.i_ppas,
             codDespesa   = planej_ppa.i_planej_ppa,
             exercicioPPA = ppas.i_exercicios, 
             isnull(metas_fin_ppa.valor, 0) as metaValorTotal,
             isnull(metas_fin_anuais_ppa.vlr_ano1, (cast((metaValorTotal/4) AS NUMERIC(15,2))), 0) as metaValorAno1,
             isnull(metas_fin_anuais_ppa.vlr_ano2, (cast((metaValorTotal/4) AS NUMERIC(15,2))), 0) as metaValorAno2,
             isnull(metas_fin_anuais_ppa.vlr_ano3, (cast((metaValorTotal/4) AS NUMERIC(15,2))), 0) as metaValorAno3,
             isnull(metas_fin_anuais_ppa.vlr_ano4, (metaValorTotal - metaValorAno1 - metaValorAno2 - metaValorAno3), 0) as metaValorAno4                  
        from bethadba.planej_ppa 
             join bethadba.hist_planej_ppa on (planej_ppa.i_ppas = hist_planej_ppa.i_ppas and 
                                               planej_ppa.i_planej_ppa = hist_planej_ppa.i_planej_ppa and 
                                               hist_planej_ppa.tipo_alteracao = 1) 
             join bethadba.ppas on (ppas.i_ppas = planej_ppa.i_ppas) 
             join bethadba.metas_fin_ppa on (metas_fin_ppa.i_ppas = hist_planej_ppa.i_ppas and
                                             metas_fin_ppa.i_planej_ppa = hist_planej_ppa.i_planej_ppa and 
                                         metas_fin_ppa.i_dt_alteracao = hist_planej_ppa.i_dt_alteracao)
         left join bethadba.metas_fin_anuais_ppa on (metas_fin_ppa.i_ppas = metas_fin_anuais_ppa.i_ppas and
                                                metas_fin_ppa.i_planej_ppa = metas_fin_anuais_ppa.i_planej_ppa and 
                                                metas_fin_ppa.i_dt_alteracao = metas_fin_anuais_ppa.i_dt_alteracao and
                                                metas_fin_ppa.i_metas_fin_ppa = metas_fin_anuais_ppa.i_metas_fin_ppa)                         
         join bethadba.contas on (metas_fin_ppa.i_planos_contas = contas.i_planos_contas and
                                  metas_fin_ppa.i_contas = contas.i_contas)
    where 2019 between ppas.i_exercicios and ppas.exerc_fin and 
         ppas.dt_aprovacao is not null and 
         metas_fin_ppa.valor > 0) as Tab
 group by entidade, exercicio, organograma, naturezaDespesa, funcao, subfuncao, programa, acao
 order by entidade, exercicio, organograma, naturezaDespesa, funcao, subfuncao, programa, acao
```